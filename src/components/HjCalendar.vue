<template>
  <div class="hj-calendar" >
    <input type="text" v-model="inputActiveDate" v-on:focus="open()" v-on:click="open()" />
    <div v-if="opened" :class="mobileClass ? mobileClass : ''" ref="hjCalendar" @click="closeOverlay">
      <section class="mounth">
        <header>
          <h3>{{getCurrentShownDateFormat('MMMM YYYY')}}</h3>
          <h5>{{getFirstShownFormat('iMMM iYYYY')}} - {{getLastShownFormat('iMMM iYYYY')}}</h5>
          <nav role='padigation'>
            <span @click="incrementMonth(-1)">&lt;</span>
            <span @click="incrementMonth(1)">></span>
          </nav>
        </header>
        <article>
          <div class="days">
            <b v-for="weekDay in weekDays" :key="weekDay">{{weekDay}}</b>
          </div>
          <div class="dates">
            <span v-for="day in currentDays" :key="day.key" class="date"
              :class="[day.outOfRange ? 'disable' : '',day.isActive ? 'active':'']" @click="onDayClick(day)">
              <h4 class="main-date">{{day.date.date()}}</h4>
              <h6 class="second-date">{{day.date.iDate()}}</h6>
            </span>
          </div>
        </article>
      </section>
    </div>
  </div>
</template>

<script>
  import moment_hijri from 'moment-hijri'

  export default {
    props: {
        value: {type: String, default: null},
        weekDays: {
            type: Array,
            default: () => ([
                'Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa', 'Su'
            ])
        },
    },
    data: function () {
      return {
        selectedMonth: 2,
        selectedYear: 2020,
        startedToday: moment_hijri(),
        currentDate: null,
        activeDate: null,
        opened: false,
        mobileWidth: 500,
        mobileClass:null,
        inputActiveDate: null,
      }
    },
    computed: {
      currentDays() {
        return this.getCurrentDays();
      },
      firstShownDay() {
        if (this.currentDays) {
          return this.currentDays[0];
        }
        return null;
      },
      lastShownDay() {
        return this.currentDays[this.currentDays.length - 1];
      },
      currentShownDate() {
        return moment_hijri().month(this.selectedMonth).year(this.selectedYear);
      },
    },
    mounted() {
      if(this.value != null){
        this.inputActiveDate = this.value
      }
      if (this.inputActiveDate != null) {
        this.activeDate = moment_hijri(this.inputActiveDate)
        if(this.activeDate.isValid()){
          this.selectedMonth = this.activeDate.month()
          this.selectedYear = this.activeDate.year()
        }
      }
      if (this.currentDate == null) {
        this.currentDate = this.currentShownDate.clone();
      }
    },
    beforeDestroy() {
      this.removeCloseEvents();
    },
    methods: {
      getCurrentDays() {
        const days = [];
        const date = this.currentShownDate.clone().startOf('month');
        const offset = 0; // it should be 1 in case of monday as start
        const startDay = date.day() || 7;
        if (startDay > (1 - offset)) {
          for (let i = startDay - (2 - offset); i >= 0; i--) {
            const prevDate = date.clone();
            prevDate.day(-i - 1);
            days.push({
              key: prevDate.valueOf(),
              outOfRange: true,
              date: prevDate
            });
          }
        }
        while (date.month() === this.currentShownDate.month()) {
          const dd = date.clone();
          let {isToday,isActive} = false
          if (dd.date() === this.startedToday.date() && dd.month() === this.startedToday.month() && dd.year() === this.startedToday.year()) {
              isToday = true;
          }
          if(this.activeDate && dd.date() === this.activeDate.date() && dd.month() === this.activeDate.month() && dd.year() === this.activeDate.year()){
              isActive = true;
          }
          days.push({
              key       : dd.valueOf(),
              date      : dd,
              isToday   : isToday,
              isActive  : isActive
          });
          date.date(date.date() + 1);
        }
        const daysLeft = 7 - days.length % 7;
        for (let i = 1; i <= daysLeft; i++) {
          const nextDate = date.clone();
          nextDate.date(nextDate.date() + i - 1);
          days.push({
            key: nextDate.valueOf(),
            outOfRange: true,
            date: nextDate
          });
        }
        return days;
      },
      getFirstShownFormat(format, lang = 'en') {
        if (this.firstShownDay) {
          return this.firstShownDay.date.locale(lang).format(format);
        }
        return '';
      },
      getLastShownFormat(format, lang = 'en') {
        if (this.lastShownDay) {
          return this.lastShownDay.date.locale(lang).format(format);
        }
        return '';
      },
      getCurrentShownDateFormat(format, lang = 'en') {
        if (this.currentShownDate) {
          return this.currentShownDate.locale(lang).format(format);
        }
        return '';
      },
      incrementMonth(increment) {
        this.currentDate.month(this.currentDate.month() + increment);
        this.selectedMonth = this.currentDate.month();
        this.selectedYear = this.currentDate.year();
        console.log(this.currentDate.year());
      },
      onDayClick(day) {
        if (!day.outOfRange) {
          this.activeDate = day.date
          this.inputActiveDate = day.date.format('YYYY-MM-DD')
          this.$emit('input',this.inputActiveDate)
          this.$emit('dateChanged',this.activeDate)
          this.close()
        }
      },
      open() {
        if (!this.opened) {
          this.opened = true;
          this.addCloseEvents();
          this.addResizeEvent();
        }
      },
      addCloseEvents() {
        if (!this.closeEventListener) {
          this.closeEventListener = e => this.keyCloseEvent(e);
          ['click', 'keyup', 'focusout'].forEach(
            eventName => document.addEventListener(eventName, this.closeEventListener)
          );
        }
      },
      closeOverlay(e) {
        if (this.mobileClass && e.target === this.$refs.hjCalendar) {
          this.close();
        }
      },
      addResizeEvent() {
          if (!this.ResizeEventListener) {
              this.ResizeEventListener = () => this.sizeChanged();
              window.addEventListener('resize', this.ResizeEventListener);
          }
          this.sizeChanged();
      },
      sizeChanged() {
        if (window.innerWidth < this.mobileWidth) {
          this.mobileClass = 'center-postion';
          return;
        }
        this.mobileClass = null;
      },
      removeCloseEvents() {
            if (this.closeEventListener) {
                ['click', 'keyup'].forEach(
                    eventName => document.removeEventListener(eventName, this.closeEventListener)
                );
                delete this.closeEventListener;
            }
      },
      removeResizeEvents() {
        if (this.ResizeEventListener) {
            this.mobileClass = undefined;
            window.removeEventListener('resize', this.ResizeEventListener);
            delete this.ResizeEventListener;
        }
      },
      keyCloseEvent(event) {
        if (event.keyCode && (event.keyCode === 27 || event.keyCode === 13)) {
          this.close();
        }else if (!(event.target === this.$el) && !this.$el.contains(event.target)) {
          this.close();
        }
      },
      close() {
        if (this.opened) {
            this.opened = false;
        }
      },
    },
    watch: {
      inputActiveDate() {
        const d = moment_hijri(this.inputActiveDate);
        if (d.isValid()) {
          this.selectedMonth = d.month()
          this.selectedYear = d.year()
          this.currentDate.month(this.selectedMonth).year(this.selectedYear)
          this.activeDate = d;
        }
      }
    }
  }
</script>

<style lang="scss">
  ::selection {
    background: rgba(0, 0, 0, .05);
  }

  body {
    padding: 20px;
    background: #d2d5e0;
    font-family: 'Open Sans', sans-serif;
  }

  ul,
  h1,
  h2,
  h3,
  h4 {
    margin: 0;
    padding: 0;
    list-style: none;
  }

  a,
  li,
  header nav span,
  article span {
    text-decoration: none;
    transition: all .25s ease-in-out;
  }

  article span.date {
    position: relative;
  }
  .hj-calendar{
    width: max-content;
  }
  .center-postion {
    position: fixed;
    left: 0;
    top: 0;
    bottom: 0;
    right: 0;
    padding: 2em;
    display: -webkit-box;
    display: flex;
    -webkit-box-pack: center;
    justify-content: center;
    -webkit-box-align: center;
    align-items: center;
    background-color: rgba(0, 0, 0, 0.3);
  } 
  article span .main-date {
    position: absolute;
    margin-top: 0px;
    margin-bottom: 0px;
    bottom: 2px;
    left: 8px;
  }

  article span .second-date {
    position: absolute;
    margin-top: 0px;
    margin-bottom: 0px;
    bottom: -10px;
    right: 3px;
    color: #ff3000;
  }

  article span.disable .second-date {
    color: #dbbbb4;
  }

  h1 {
    font: 300 28px 'Open Sans';
    color: #4c5373;
  }

  /****/
  /* Left Bar */
  /****/
  nav.sections {
    overflow: hidden;
    display: block;
    float: left;
    width: 82px;
    height: auto;
    margin-right: 10px;
    background: #3bc1f8;
    box-shadow: 0 1px 1px rgba(78, 200, 247, .35);
    border-radius: 5px;
    text-aling: center;
  }

  nav.sections li a {
    display: block;
    width: 100%;
    height: 82px;
  }

  nav.sections li:hover,
  nav.sections li.active {
    background: #00aced;
    box-shadow: inset 0 1px 1px rgba(0, 0, 0, .07);
  }


  nav.sections li.notes a {
    background-position: 25px 25px;
  }

  nav.sections li.calendar a {
    background-position: 25px -57px;
  }

  nav.sections li.settings a {
    background-position: 25px -139px;
  }

  /****/
  /* Calendar */
  /****/
  section.mounth {
    overflow: hidden;
    display: block;
    width: 336px;
    height: auto;
    background: #fff;
    box-shadow: 0 1px 1px rgba(0, 0, 0, .25);
    border-radius: 5px;
  }

  header {
    position: relative;
    display: block;
    height: 70px;
    background: #e9eaf0;
    box-shadow: inset 0 -1px 0 rgba(0, 0, 0, .06);
  }

  header h3 {
    margin-left: 20px;
    margin-bottom: 0px;
    line-height: 40px;
  }

  header h5 {
    margin-left: 20px;
    margin-top: 0px;
    color: #ff3000;
  }

  header nav {
    overflow: hidden;
    position: absolute;
    display: block;
    right: 30px;
    top: 30px;
  }

  header nav span {
    display: block;
    float: left;
    width: 11px;
    height: 20px;
    margin-left: 30px;
    cursor: pointer;
    //background: url('http://ysfu.net/sprite.png') no-repeat;
  }

  header nav span:first-of-type {
    background-position: -77px -4px;
  }

  header nav span:first-of-type:hover {
    background-position: -106px -4px;
  }

  header nav span:last-of-type {
    background-position: -90px -4px;
  }

  header nav span:last-of-type:hover {
    background-position: -119px -4px;
  }

  article b,
  article span {
    display: block;
    float: left;
    width: 40px;
    height: 40px;
    margin: 4px;
    text-align: center;
  }

  article b {
    font: 600 16px/45px 'Open Sans';
    color: #91d9ff;
  }

  article span {
    font: 300 19px/45px 'Open Sans';
    color: #4c5373;
    box-shadow: inset 0 0 1px 1px #d9dce5;
    border-radius: 2px;
    cursor: pointer;
  }

  article span.disable {
    color: #b6bacc;
    box-shadow: inset 0 0 1px 1px #f3f4f7;
  }

  article span:not(.disable):hover {
    background: #cdd0dd;
    box-shadow:
      inset 0 0 1px 1px #d9dce5,
      0 1px 3px -1px #cdd0dd;
  }

  article span:not(.disable):active {
    background: #b9becf;
    box-shadow:
      inset 0 1px 2px rgba(0, 0, 0, .18),
      0 0 0 #cdd0dd;
  }

  article span.active,
  article span.active:hover,
  article span.active:active {
    color: #fff;
    box-shadow:
      inset 0 0 1px 1px #ff8263,
      0 1px 3px -1px #ff8263;
    background: #ff8263;
  }

  article div {
    display: block;
    overflow: hidden;
    clear: both;
  }
</style>