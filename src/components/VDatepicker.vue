<template>
    <div>
        <div class="cont dpicker-fields">
            <input type="text"
                   placeholder="Check In"
                   v-model="checkIn"
                   :class="{ 'chosen-dpicker-field': !isCheckOut && isCheckOut !== null }"
                   v-on:click="showCalendar($event, false)"
            >
            <input type="text"
                   placeholder="Check Out"
                   v-model="checkOut"
                   :class="{ 'chosen-dpicker-field': isCheckOut }"
                   v-on:click="showCalendar($event, true)"
            >
        </div>
        <div id="sham-datepicker" v-if="seen" v-on:mouse1leave="hideCalendar">
            <div class="h-wrapper d-grid">
                <a @click="decreaseMonth">
                    Left
                </a>
                <div>{{ currentMonth.monthName }} {{ currentMonth.year }}</div>
                <a @click="increaseMonth">
                    Right
                </a>
            </div>
            <hr/>
            <ul>
                <li>{{ mouseFocusedDate }}</li>
            </ul>
            <div class="w-wrapper d-grid">
                <div v-for="dayn in weeks" class="g-item">{{ dayn }}</div>
            </div>
            <hr/>
            <div class="w-wrapper d-grid">
                <a
                    v-for="month, index in adjacentMonthRange"
                    class="g-item"
                    :class="{
                        'picked-day': isDayPicked(month),
                        'in-range-day': isDayInRange(month)
                    }"
                    v-on:mouseover="initRange($event, index)"
                    v-on:click="selectDate($event, month)"
                >{{ month.day }}
                    <div :style="{ display: 'none' }"
                    >{{ convertRangeDayInstanceToClassName(month, false, '-') }}
                    </div>
                </a>
            </div>
            <div>
                <a
                    class="dp-clear-dates-btn"
                    v-on:click="clearDates">
                    Clear dates
                </a>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: "VDatepicker",
        data: function() {
            return {
                checkIn: 'Check In',
                checkOut: 'Check Out',
                seen: true,
                isCheckOut: null,
                weeks: ['SU','MO','TU','WE','TH','Fr','SA'],
                currentDate: 10,
                days: Array(42).fill('*'),
                bindedText: '',
                hoveredDate: '',
                mouseFocusedDate: null
            }
        },
        computed: {
            currentMonth: function() {
                return this.getMonthFromNum(this.currentDate)
            },
            adjacentMonthsDayBegin: function() {
                let centerDate = this.currentMonth.fullDate;

                let dateRange = Array(3).fill(1).map((elem,index) => {
                    let date = new Date(centerDate);
                    date.setMonth(date.getMonth() + index - 1);
                    let firstDayOfMonth = new Date(date.getFullYear(), date.getMonth(), 1).getDay();

                    return firstDayOfMonth;
                });
                return dateRange;
            },
            adjacentMonthsDayEnd: function() {
                let centerDate = this.currentMonth.fullDate;

                let dateRange = Array(3).fill(1).map((elem,index) => {
                    let date = new Date(centerDate);
                    date.setMonth(date.getMonth() + index - 1);
                    let lastDayOfMonth = new Date(date.getFullYear(), date.getMonth()+1, 0).getDate();

                    return lastDayOfMonth;
                });

                return dateRange;
            },
            adjacentMonthRange: function() {
                let main = {
                    range: []
                };

                function RangeDay(day, month, year) {
                    this.day = day;
                    this.month = month;
                    this.year = year;

                    return {
                        day: this.day,
                        month: this.month,
                        year: this.year
                    }
                }

                let [start, end] = [this.adjacentMonthsDayBegin, this.adjacentMonthsDayEnd];
                let prevRange = [],
                    nextRange = [];
                let dateYear = this.currentMonth.year;

                for(let i = 1;i < 43; i++) {
                    if(i >= start[1] && i < end[1] + start[1]) {
                        main.range.push(new RangeDay(i - start[1] + 1, this.currentMonth.monthNumber, dateYear));
                    } else {
                        let yearOffset = 0;

                        if(i < start[1]) {
                            if(this.currentMonth.monthNumber === 0) yearOffset = -1;

                            prevRange.push(new RangeDay(Math.abs(end[0] - i + 1), this.currentMonth.monthNumber - 1, dateYear + yearOffset));
                        } else {
                            if(this.currentMonth.monthNumber === 11) yearOffset = 1;

                            nextRange.push(new RangeDay(Math.abs(i - start[1] - end[1] + 1), this.currentMonth.monthNumber + 1, dateYear + yearOffset));
                        }
                    }
                }

                prevRange.reverse();

                return [...prevRange, ...main.range, ...nextRange];
            }
        },
        methods: {
            initRange: function($event, index) {
                let internalDate = $event.target.firstElementChild.innerText;
                this.mouseFocusedDate = internalDate.replace(/(\s|\n)/g, '');
                let firstComparison = this.compareTwoDates(this.checkIn, this.mouseFocusedDate);
                let secondComparison = this.compareTwoDates(this.checkOut, this.mouseFocusedDate);

                if(!this.isCheckOut) {

                } else {

                }
            },
            isDayInRange: function() {

            },
            clearDates: function() {
                this.checkIn = 'Check In';
                this.checkOut = 'Check Out';
            },
            isDayPicked: function(month) {
                let equalsTo;

                if(!this.isCheckOut) {
                    equalsTo = this.checkOut;
                } else {
                    equalsTo = this.checkIn;
                }

                return this.convertRangeDayInstanceToClassName(month, true, '/') === equalsTo;
            },
            selectDate: function(event, rangeDayInstance) {
                let selectedDate = this.convertRangeDayInstanceToClassName(
                    rangeDayInstance, true, '/');
                let incrementBy = 0;
                let fieldsInvalid = {
                    checkIn: this.checkIn === 'Check In',
                    checkOut: this.checkOut === 'Check Out'
                };

                if(this.isCheckOut) {
                    this.checkOut = selectedDate;
                    incrementBy = -1;
                } else {
                    this.checkIn = selectedDate;
                    incrementBy = 1;
                }

                let dateComparison = this.compareTwoDates(this.checkIn, this.checkOut);

                if(dateComparison.firstBigger || dateComparison.equals) {
                    let consideredDate;

                    !this.isCheckOut ? consideredDate = this.checkIn : consideredDate = this.checkOut;
                    let incremented = this.incrementDateBy(consideredDate, 'day', incrementBy);
                    incremented = this.getDateConverted(incremented, true, '/');
                    this.isCheckOut ? this.checkIn = incremented : this.checkOut = incremented;
                }

                if(fieldsInvalid.checkIn || fieldsInvalid.checkOut) {
                    this.isCheckOut = !this.isCheckOut;
                }

                this.hoveredDate = event.target.firstElementChild.innerText;
            },
            showCalendar: function(event, isCheckOut) {
                this.isCheckOut = isCheckOut;
                this.seen = true;
            },
            hideCalendar: function(event) {
                this.seen = false;
            },
            computeMonth: function(date) {
                return (Math.random() * date).toFixed(1)
            },
            increaseMonth: function() {
                this.currentDate++;
            },
            decreaseMonth: function() {
                this.currentDate--;
            },
            getMonthFromNum: function(num) {
                let date = new Date();
                date.setMonth(num);
                let locale = "en-us";
                let month = date.toLocaleString(locale, { month: "long" });

                return {
                    monthName: month,
                    year: date.getFullYear(),
                    fullDate: date,
                    monthNumber: date.getMonth()
                }
            },
            getDateConverted: function (date, sliceYear, delimiter = '-') {
                // mm-dd-yy
                let current = new Date(date);
                let converted = [
                    current.getMonth() + 1,
                    current.getDate(),
                    current.getFullYear()
                ].map((elem, index) => this.concatZeroToDate(elem, index, sliceYear));

                return converted.join(delimiter);
            },
            /* Method is used for checking inside array mapping */
            concatZeroToDate: function(date, index, sliceYear) {
                if (index !== 2 && date < 10) {
                    date = '0' + date;
                } else if (index === 2) {
                    date = date.toString().slice(0, 4);
                    if(sliceYear) date = date.slice(2);
                }

                return date;
            },
            incrementDateBy: function(initDate, param, incrementValue) {
                let incrementMethods = {
                    day: function(dt) {
                        dt = new Date(dt);
                        dt.setDate(dt.getDate() + incrementValue);
                        return dt;
                    },
                    month: function(dt) {
                        dt = new Date(dt);
                        dt.setMonth(dt.getMonth() + incrementValue);
                        return dt;
                    }
                };

                if(Object.keys(incrementMethods).indexOf(param) !== -1) {
                    let date = incrementMethods[param](initDate);
                    return date;
                } else {
                    return param;
                }
            },
            compareTwoDates: function(firstDate, secondDate) {
                // Date is supported in any format
                let dates = [...arguments].map(date => {
                    let dt = new Date(date);
                    return dt.getTime();
                });

                let info = [[...arguments], dates, dates[0] > dates[1]];

                return {
                    firstBigger: dates[0] > dates[1],
                    equals: dates[0] === dates[1],
                    lastBigger: dates[0] < dates[1]
                };
            },
            convertDateToInput: function(date) {
                let splittedDate = this.getDateConverted(date).split('-');

                if(splittedDate[2].length > 2) {
                    splittedDate[2] = splittedDate.slice(2)
                };
                return splittedDate.join('/');
            },
            convertRangeDayInstanceToClassName: function(rangeDayInstance, sliceYear, delimiter = '-') {
                /*
              * Please take into consideration that month value is incremented by 1
              */
                let splittedDate = [
                    rangeDayInstance.month + 1,
                    rangeDayInstance.day,
                    rangeDayInstance.year,
                ].map((date, index) => this.concatZeroToDate(date, index));

                if(sliceYear) splittedDate[2] = splittedDate[2].toString().slice(2);

                return splittedDate.join(delimiter);
            }
        }
    }
</script>

<style lang="scss">

    $bg-chosen-day: #52A2FF;
    $range-opacity: 0.35;
    $dp-width: 320px;
    $dp-height: 332px + 25;

    .cont {
        padding: 15px;
    }

    .dpicker-fields {
        input {
            border: none;
            border-radius: 6px;
            width: 142px;
            padding: 8px;
            font-size: 16px;
        }

        .chosen-dpicker-field {
            color: $bg-chosen-day;
            background: #a3cdff;
        }
    }

    #sham-datepicker {
        font-family: 'Open Sans';
        font-weight: 600;
        position: absolute;
        width: $dp-width;
        height: $dp-height;
        background: #ffffff;
        box-shadow: 0 6px 32px 0 rgba(0,0,0,0.15);
        border-radius: 6px;
        left: 15px;
        margin: 0 auto;

        hr {
            margin: 0px;
            opacity: 0.5;
            height: 1px;
            background: #EDEDED;
        }

        a {
            cursor: pointer;
        }

        .d-grid {
            display: grid;
        }

        .h-wrapper {
            padding: 13px 20px 13px;
            grid-template-columns: 40px 1fr 40px;
            text-align: center;
        }

        .dp-clear-dates-btn {
            font-size: 13px;
            color: $bg-chosen-day;
            float: right;
            margin-right: 45px;

            &:hover {
                text-decoration: underline;
            }
        }

        .w-wrapper {
            padding: 5px 30px;
            /* padding: 7.5px 30px; */
            grid-template-columns: repeat(7, 1fr);
            /* grid-column-gap: 5px; */
            grid-row-gap: 5px;

            a {
                cursor: pointer;
            }

            .g-item {
                text-align: center;
                justify-item: center;
                /* background: #ddd; */
                background: #fff;
                padding: 5px 0 5px;
                width: 100%;
            }

            .picked-day {
                background: $bg-chosen-day;
                color: #fff;
                border-radius: 6px;
            }

            .hidden-bg-day {

            }

            .in-range-day {
                background: $bg-chosen-day;
                color: #fff;
                opacity: 0.35;
            }

            .in-range-first-day {
                background: $bg-chosen-day;
                color: #fff;
                border-radius: 6px 0 0 6px;
                opacity: 0.35;
            }

            .in-range-last-day {
                background: $bg-chosen-day;
                color: #fff;
                border-radius: 0 6px 6px 0;
                opacity: 0.35;
            }
        }
    }


</style>