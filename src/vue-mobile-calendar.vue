<template>
    <div class="calendar" v-show="show">
        <div class="change-month">
            <div class="change-item year" v-if="change">
                <span class="change-reduce" @click="reduceYear"><i></i></span>
                <span>{{date.getFullYear()}}年</span>
                <span class="change-plus" @click="plusYear"><i></i></span>
            </div>
            <div class="current-month" v-if="month">{{date.getFullYear() + '年' + (date.getMonth() + 1) + '月'}}</div>
            <div class="change-item month" v-if="change">
                <span class="change-reduce" @click="reduceMonth"><i></i></span>
                <span>{{date.getMonth() + 1}}月</span>
                <span class="change-plus" @click="plusMonth"><i></i></span>
            </div>
        </div>
        <div class="calendar-table">
            <div class="t-head">
                <div class="tr week-day">
                    <div class="td weekend">日</div>
                    <div class="td">一</div>
                    <div class="td">二</div>
                    <div class="td">三</div>
                    <div class="td">四</div>
                    <div class="td">五</div>
                    <div class="td weekend">六</div>
                </div>
            </div>
            <div class="area" :style="{height: `${height}px`}">
                <div class="month" :class="{animation: moveInfo.animation}" :style="{transform: `translateY(${moveInfo.y}px)`}" @touchstart="moveStart" @touchmove="moving" @touchend="moveEnd">
                    <div class="prev">
                        <div class="t-body" v-for="(m, num) in months.prev" :ref="`prev_${num}`">
                            <div class="tr" v-for="(tr, index) in m">
                                <div class="td" v-for="(td, i) in tr" :class="{weekend: td.weekday == 0 || td.weekday == 6, enable: td.enable, 'out-month': !td.inMonth}"
                                     @click="selectDate(index, i)">
                                    <div class="date-info" :class="{now: td.isToday, select: selected == td.dateStr}">
                                        <div class="date-num">{{td.day}}</div>
                                        <div class="date-lunar" v-if="lunar">{{td.lunar.showInLunar}}</div>
                                        <div class="sch-icon" v-if="td.sch"></div>
                                    </div>
                                    <slot v-bind:date="td"></slot>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="t-body current" ref="current">
                        <div class="tr" v-for="(tr, index) in months.current">
                            <div class="td" v-for="(td, i) in tr" :class="{weekend: td.weekday == 0 || td.weekday == 6, enable: td.enable, 'out-month': !td.inMonth}"
                                 @click="selectDate(index, i)">
                                <div class="date-info" :class="{now: td.isToday, select: selected == td.dateStr}">
                                    <div class="date-num">{{td.day}}</div>
                                    <div class="date-lunar" v-if="lunar">{{td.lunar.showInLunar}}</div>
                                    <div class="sch-icon" v-if="td.sch"></div>
                                </div>
                                <slot v-bind:date="td"></slot>
                            </div>
                        </div>
                    </div>
                    <div class="next">
                        <div class="t-body" v-for="(m, num) in months.next" :ref="`next_${num}`">
                            <div class="tr" v-for="(tr, index) in m">
                                <div class="td" v-for="(td, i) in tr" :class="{weekend: td.weekday == 0 || td.weekday == 6, enable: td.enable, 'out-month': !td.inMonth}"
                                     @click="selectDate(index, i)">
                                    <div class="date-info" :class="{now: td.isToday, select: selected == td.dateStr}">
                                        <div class="date-num">{{td.day}}</div>
                                        <div class="date-lunar" v-if="lunar">{{td.lunar.showInLunar}}</div>
                                        <div class="sch-icon" v-if="td.sch"></div>
                                    </div>
                                    <slot v-bind:date="td"></slot>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: "VueMobileCalendar",
        props: {
            selected: {
                type: String,
                default: null
            },
            show: {
                type: Boolean,
                default: true
            },
            //是否显示日历月份开关
            month: {
                type: Boolean,
                default: true
            },
            //是否显示日历月份手动调整开关
            change: {
                type: Boolean,
                default: false
            },
            lunar: {
                type: Boolean,
                default: false
            },
            init: {
                type: Boolean,
                default: true
            },
            //左右滑动切换月份开关
            touch: {
                type: Boolean,
                default: false
            }
        },
        data() {
            return {
                months: {
                    prev: [],
                    current: [],
                    next: []
                },
                moveInfo: {
                    start: null,
                    y: 0,
                    position: 0,
                    moved: false,
                    animation: false
                },
                height: 0,
                date: null
            }
        },
        created: function () {
            if (this.selected) {
                this.date = new Date(this.selected.slice(0, -2) + "01");
            } else {
                this.date = new Date(new Date().setDate(1));
            }
        },
        mounted: function () {
            if (this.init) {
                this.months.current = this.getDateArray();
                this.$nextTick(() => {
                    let height = this.$el.querySelector(".month").clientHeight;
                    this.height = height;
                });
            }
            this.$nextTick(() => {
            })
        },
        methods: {
            reduceYear: function () {
                this.date = new Date(this.date.setYear(this.date.getFullYear() - 1));
                this.getDateArray();
            },
            plusYear: function () {
                this.date = new Date(this.date.setYear(this.date.getFullYear() + 1));
                this.getDateArray();
            },
            reduceMonth: function () {
                this.date = new Date(this.date.setMonth(this.date.getMonth() - 1));
                this.getDateArray();
            },
            plusMonth: function () {
                this.date = new Date(this.date.setMonth(this.date.getMonth() + 1));
                this.getDateArray();
            },
            selectDate: function (index, i) {
                let date = this.months.current[index][i];
                this.$emit("change", date);
            },
            moveStart: function (e) {
                let dom = e.currentTarget;
                this.moveInfo.moved = false;
                this.moveInfo.animation = false;
                this.moveInfo.start = e.targetTouches[0].pageY;
                this.moveInfo.position = (dom.style.WebkitTransform.replace(/translateY\(/g, "").replace(/px\)/g, "")) * 1;
            },
            moving: function (e) {
                let distance = e.targetTouches[0].pageY - this.moveInfo.start;
                if (distance == 0 && !this.moved) {
                    return;
                }
                e.preventDefault();
                this.moveInfo.moved = true;
                let offset = distance + this.moveInfo.position;
                let current = new Date(this.date.getFullYear(), this.date.getMonth(), this.date.getDate());
                if(distance > 0) {
                    let height = this.$el.querySelector('.prev').clientHeight;
                    //向下滑动超出已有日历范围，构建上个月的日历
                    if(offset > height) {
                        let month = new Date(current.setMonth(this.date.getMonth() - 1));
                        this.months.prev.unshift(this.getDateArray(month));
                    }
                } else if(distance < 0) {
                    let height = this.$el.querySelector('.next').clientHeight;
                    //向上滑动超出已有日历范围，构建下个月的日历
                    if(offset < -1 * height) {
                        let month = new Date(current.setMonth(this.date.getMonth() + 1));
                        this.months.next.push(this.getDateArray(month));
                    }
                }
                this.moveInfo.y = offset;
            },
            moveEnd: function (e) {
                if (!this.moveInfo.moved) {
                    return;
                }
                e.preventDefault();
                let y = this.moveInfo.y;
                //滚动到初始位置不做任何响应
                if(y == 0) { return; }
                let domList = this.$refs;
                let current = domList.current;
                let unit = current.querySelector(".td").clientHeight;
                //向上滚动还是向下滚动
                let key = y > 0 ? 'prev' : 'next';
                let coe = y > 0 ? 1 : -1;
                let list = this.months[key];
                let index, h = 0;
                if (y > 0) {
                    //根据滚动位置计算当前滚动到哪个月份
                    for(let i = 0, l = list.length; i < l; i++) {
                        let height = domList[`${key}_${i}`][0].clientHeight;
                        let offset = h + height - y;
                        if(offset > 0) {
                            let distance = y - h;
                            if(distance >= unit) {
                                index = i;
                                this.moveInfo.y = (h + height) * coe;
                                this.moveInfo.animation = true;
                                this.height = height;
                            } else if(distance > 0) {
                                index = i - 1;
                                if(i - 1 > 0) {
                                    this.height = domList[`${key}_${index}`][0].clientHeight;
                                }
                                this.moveInfo.y = h * coe;
                                this.moveInfo.animation = true;
                            }
                            setTimeout(() => {
                                this.moveInfo.animation = false;
                                let temp = (index + 1) * coe;
                                this.date.setMonth(this.date.getMonth() - temp);
                                temp != 0 && this.$emit('move', this.date);
                                let last = index == -1 ? this.months.current : list[index];
                                this.$nextTick(() => {
                                    this.moveInfo.y = 0;
                                    this.months = {
                                        prev: [],
                                        current: last,
                                        next: []
                                    }
                                });
                            }, 300);
                            break;
                        } else {
                            h += height;
                            continue;
                        }
                    }
                } else {
                    y = Math.abs(y);
                    for(let j = 0, l = list.length; j < l; j++) {
                        let height = domList[`${key}_${j}`][0].clientHeight;
                        let add = j == 0 ? current.clientHeight :  domList[`${key}_${j - 1}`][0].clientHeight;
                        if(y > h && y < h + add) {
                            let distance = y - h;
                            if(distance >= unit) {
                                index = j;
                                this.moveInfo.y = (h + add) * coe;
                                this.moveInfo.animation = true;
                                this.height = height;
                            } else if(distance > 0) {
                                index = j - 1;
                                if(j - 1 > 0) {
                                    this.height = domList[`${key}_${index}`][0].clientHeight;;
                                }
                                this.moveInfo.y = h * coe;
                                this.moveInfo.animation = true;
                            }
                            setTimeout(() => {
                                this.moveInfo.animation = false;
                                let temp = (index + 1) * coe;
                                this.date.setMonth(this.date.getMonth() - temp);
                                temp != 0 && this.$emit('move', this.date);
                                let last = index == -1 ? this.months.current : list[index];
                                this.$nextTick(() => {
                                    this.moveInfo.y = 0;
                                    this.months = {
                                        prev: [],
                                        current: last,
                                        next: []
                                    }
                                });
                            }, 300);
                            break;
                        } else {
                            h += height;
                            continue;
                        }
                    }
                }
            },
            getDateArray: function (time) {
                time = time || this.date;
                let year = time.getFullYear();
                let month = time.getMonth();
                let firstDay = new Date(year, month, 1); //当月第一天
                let weekDay = firstDay.getDay(); //当月第一天是星期几
                let dayArray = this.getMonthDays();
                let weekNum = Math.ceil((dayArray[month] + weekDay) / 7); //当月显示周数
                let trList = [];
                for (let i = 0; i < weekNum; i++) {
                    let tdList = [];
                    for (let j = 0; j < 7; j++) {
                        let index = i * 7 + j; //表格单元的自然序号
                        let date = index - weekDay + 1; //计算日期
                        let dateObj, ty, tm, td;
                        let inMonth = false;
                        if (date <= 0) {
                            tm = month - 1 < 0 ? 11 : month - 1;
                            ty = month - 1 < 0 ? year - 1 : year;
                            td = dayArray[tm] + date;
                        } else if (date > dayArray[month]) {
                            tm = month + 1 > 11 ? 0 : month + 1;
                            ty = month + 1 > 11 ? year + 1 : year;
                            td = date - dayArray[month];
                        } else {
                            ty = year;
                            tm = month;
                            td = date;
                            inMonth = true;
                        }
                        let dateStr = this.getDateString(ty, tm, td);
                        dateObj = {
                            weekday: j,
                            day: td,
                            isToday: this.isToday(ty, tm, td),
                            dateStr: dateStr,
                            inMonth
                        };
                        this.lunar && (dateObj.lunar = this.getLunarDay(new Date(ty, tm, td)));
                        tdList.push(dateObj);
                    }
                    trList.push(tdList);
                }
                return trList;
            },
            isToday: function (ty, tm, td) {
                let d = new Date();
                return d.getFullYear() == ty && d.getMonth() == tm && d.getDate() == td;
            },
            getDateString: function (year, month, date) {
                return year + "-" + ((month + 1) < 10 ? ("0" + (month + 1)) : (month + 1)) + "-" + (date < 10 ? ("0" + date) : date);
            },
            isLeap: function (year) {
                return (year % 100 == 0 ? (year % 400 == 0 ? 1 : 0) : (year % 4 == 0 ? 1 : 0));
            },
            getMonthDays: function (year) {
                return new Array(31, 28 + this.isLeap(year), 31, 30, 31, 30, 31, 31, 30, 31, 30, 31);
            },
            getLunarDay: function (date) {
                let lunarInfo = [19416, 19168, 42352, 21717, 53856, 55632, 91476, 22176, 39632, 21970, 19168, 42422, 42192, 53840, 119381, 46400, 54944, 44450, 38320, 84343, 18800, 42160, 46261, 27216, 27968, 109396, 11104, 38256, 21234, 18800, 25958, 54432, 59984, 28309, 23248, 11104, 100067, 37600, 116951, 51536, 54432, 120998, 46416, 22176, 107956, 9680, 37584, 53938, 43344, 46423, 27808, 46416, 86869, 19872, 42448, 83315, 21200, 43432, 59728, 27296, 44710, 43856, 19296, 43748, 42352, 21088, 62051, 55632, 23383, 22176, 38608, 19925, 19152, 42192, 54484, 53840, 54616, 46400, 46496, 103846, 38320, 18864, 43380, 42160, 45690, 27216, 27968, 44870, 43872, 38256, 19189, 18800, 25776, 29859, 59984, 27480, 21952, 43872, 38613, 37600, 51552, 55636, 54432, 55888, 30034, 22176, 43959, 9680, 37584, 51893, 43344, 46240, 47780, 44368, 21977, 19360, 42416, 86390, 21168, 43312, 31060, 27296, 44368, 23378, 19296, 42726, 42208, 53856, 60005, 54576, 23200, 30371, 38608, 19415, 19152, 42192, 118966, 53840, 54560, 56645, 46496, 22224, 21938, 18864, 42359, 42160, 43600, 111189, 27936, 44448],
                    sTermInfo = [0, 21208, 43467, 63836, 85337, 107014, 128867, 150921, 173149, 195551, 218072, 240693, 263343, 285989, 308563, 331033, 353350, 375494, 397447, 419210, 440795, 462224, 483532, 504758];
                let Gan = "甲乙丙丁戊己庚辛壬癸", Zhi = "子丑寅卯辰巳午未申酉戌亥", Animals = "鼠牛虎兔龙蛇马羊猴鸡狗猪";
                let solarTerm = ["小寒", "大寒", "立春", "雨水", "惊蛰", "春分", "清明", "谷雨", "立夏", "小满",
                    "芒种", "夏至", "小暑", "大暑", "立秋", "处暑", "白露", "秋分", "寒露", "霜降", "立冬", "小雪", "大雪", "冬至"];
                let nStr1 = "日一二三四五六七八九十", nStr2 = "初十廿卅",
                    nStr3 = ["正", "二", "三", "四", "五", "六", "七", "八", "九", "十", "十一", "腊"],
                    sFtv1 = {
                        "0101": "*1元旦节", "0202": "湿地日",
                        "0214": "情人节", "0308": "妇女节",
                        "0312": "植树节", "0315": "消费者权益日",
                        "0401": "愚人节", "0422": "地球日",
                        "0501": "*1劳动节", "0504": "青年节",
                        "0512": "护士节", "0518": "博物馆日",
                        "0520": "母亲节", "0601": "儿童节",
                        "0623": "奥林匹克日", "0630": "父亲节",
                        "0701": "建党节", "0801": "建军节",
                        "0903": "抗战胜利日", "0910": "教师节",
                        "1001": "*3国庆节", "1201": "艾滋病日",
                        "1224": "平安夜", "1225": "圣诞节"
                    },
                    sFtv2 = {
                        "0100": "除夕", "0101": "*2春节",
                        "0115": "元宵节", "0505": "*1端午节",
                        "0707": "七夕节", "0715": "中元节",
                        "0815": "*1中秋节", "0909": "*1重阳节",
                        "1015": "下元节", "1208": "腊八节",
                        "1223": "小年"

                    };

                function flunar(Y) {
                    let sTerm = function (j, i) {
                            let h = new Date((31556925974.7 * (j - 1900) + sTermInfo[i] * 60000) + Date.UTC(1900, 0, 6, 2, 5));
                            return (h.getUTCDate())
                        },
                        d = function (k) {
                            let h, j = 348;
                            for (h = 32768; h > 8; h >>= 1) {
                                j += (lunarInfo[k - 1900] & h) ? 1 : 0;
                            }
                            return (j + b(k))
                        },
                        ymdCyl = function (h) {
                            return (Gan.charAt(h % 10) + Zhi.charAt(h % 12))
                        },
                        b = function (h) {
                            let islp = (g(h)) ? ((lunarInfo[h - 1900] & 65536) ? 30 : 29) : (0);
                            return islp
                        },
                        g = function (h) {
                            return (lunarInfo[h - 1900] & 15)
                        },
                        e = function (i, h) {
                            return ((lunarInfo[i - 1900] & (65536 >> h)) ? 30 : 29)
                        },
                        newymd = function (m) {
                            let k, j = 0, h = 0, l = new Date(1900, 0, 31), n = (m - l) / 86400000;
                            this.dayCyl = n + 40;
                            this.monCyl = 14;
                            for (k = 1900; k < 2050 && n > 0; k++) {
                                h = d(k);
                                n -= h;
                                this.monCyl += 12;
                            }
                            if (n < 0) {
                                n += h;
                                k--;
                                this.monCyl -= 12;
                            }
                            this.year = k;
                            this.yearCyl = k - 1864;
                            j = g(k);
                            this.isLeap = false;
                            for (k = 1; k < 13 && n > 0; k++) {
                                if (j > 0 && k == (j + 1) && this.isLeap == false) {
                                    --k;
                                    this.isLeap = true;
                                    h = b(this.year);
                                } else {
                                    h = e(this.year, k);
                                }
                                if (this.isLeap == true && k == (j + 1)) {
                                    this.isLeap = false;
                                }
                                n -= h;
                                if (this.isLeap == false) this.monCyl++;
                            }
                            if (n == 0 && j > 0 && k == j + 1) {
                                if (this.isLeap) {
                                    this.isLeap = false;
                                } else {
                                    this.isLeap = true;
                                    --k;
                                    --this.monCyl;
                                }
                            }
                            if (n < 0) {
                                n += h;
                                --k;
                                --this.monCyl
                            }
                            this.month = k;
                            this.day = n + 1;
                        },
                        digit = function (num) {
                            return num < 10 ? "0" + (num | 0) : num;
                        },
                        reymd = function (i, j) {
                            let h = i;
                            return j.replace(/dd?d?d?|MM?M?M?|yy?y?y?/g, function (k) {
                                switch (k) {
                                    case "yyyy":
                                        let l = "000" + h.getFullYear();
                                        return l.substring(l.length - 4);
                                    case "dd":
                                        return digit(h.getDate());
                                    case "d":
                                        return h.getDate().toString();
                                    case "MM":
                                        return digit((h.getMonth() + 1));
                                    case "M":
                                        return h.getMonth() + 1;
                                }
                            })
                        },
                        lunarMD = function (i, h) {
                            let j;
                            switch (i, h) {
                                case 10:
                                    j = "初十";
                                    break;
                                case 20:
                                    j = "二十";
                                    break;
                                case 30:
                                    j = "三十";
                                    break;
                                default:
                                    j = nStr2.charAt(Math.floor(h / 10));
                                    j += nStr1.charAt(h % 10);
                            }
                            return (j)
                        };
                    this.isToday = false;
                    this.isRestDay = false;
                    this.solarYear = reymd(Y, "yyyy");
                    this.solarMonth = reymd(Y, "M");
                    this.solarDate = reymd(Y, "d");
                    this.solarWeekDay = Y.getDay();
                    this.inWeekDays = "星期" + nStr1.charAt(this.solarWeekDay);
                    let X = new newymd(Y);
                    this.lunarYear = X.year;
                    this.shengxiao = Animals.charAt((this.lunarYear - 4) % 12);
                    this.lunarMonth = X.month;
                    this.lunarIsLeapMonth = X.isLeap;
                    this.lnongMonth = this.lunarIsLeapMonth ? "闰" + nStr3[X.month - 1] : nStr3[X.month - 1];
                    this.lunarDate = X.day;
                    this.showInLunar = this.lnongDate = lunarMD(this.lunarMonth, this.lunarDate);
                    if (this.lunarDate == 1) {
                        this.showInLunar = this.lnongMonth + "月";
                    }
                    this.ganzhiYear = ymdCyl(X.yearCyl);
                    this.ganzhiMonth = ymdCyl(X.monCyl);
                    this.ganzhiDate = ymdCyl(X.dayCyl++);
                    this.jieqi = "";
                    this.restDays = 0;
                    if (sTerm(this.solarYear, (this.solarMonth - 1) * 2) == reymd(Y, "d")) {
                        this.showInLunar = this.jieqi = solarTerm[(this.solarMonth - 1) * 2];
                    }
                    if (sTerm(this.solarYear, (this.solarMonth - 1) * 2 + 1) == reymd(Y, "d")) {
                        this.showInLunar = this.jieqi = solarTerm[(this.solarMonth - 1) * 2 + 1];
                    }
                    if (this.showInLunar == "清明") {
                        this.showInLunar = "清明节";
                        this.restDays = 1;
                    }
                    this.solarFestival = sFtv1[reymd(Y, "MM") + reymd(Y, "dd")];
                    if (typeof this.solarFestival == "undefined") {
                        this.solarFestival = "";
                    } else {
                        if (/\*(\d)/.test(this.solarFestival)) {
                            this.restDays = parseInt(RegExp.$1);
                            this.solarFestival = this.solarFestival.replace(/\*\d/, "");
                        }
                    }
                    this.showInLunar = (this.solarFestival == "") ? this.showInLunar : this.solarFestival;
                    this.lunarFestival = sFtv2[this.lunarIsLeapMonth ? "00" : digit(this.lunarMonth) + digit(this.lunarDate)];
                    if (typeof this.lunarFestival == "undefined") {
                        this.lunarFestival = "";
                    } else {
                        if (/\*(\d)/.test(this.lunarFestival)) {
                            this.restDays = (this.restDays > parseInt(RegExp.$1)) ? this.restDays : parseInt(RegExp.$1);
                            this.lunarFestival = this.lunarFestival.replace(/\*\d/, "");
                        }
                    }
                    if (this.lunarMonth == 12 && this.lunarDate == e(this.lunarYear, 12)) {
                        this.lunarFestival = sFtv2["0100"];
                        this.restDays = 1;
                    }
                    this.showInLunar = (this.lunarFestival == "") ? this.showInLunar : this.lunarFestival;
                }

                return new flunar(date);
            }
        }
    }
</script>

<style lang="less">
    @import "./less/base";
    .change-month {
        display: flex;
        align-items: center;
        justify-content: space-between;
    }
    .current-month {
        flex: 1;
        margin-top: 12px;
        font-size: 18px;
        line-height: 26px;
        font-weight: 600;
        text-align: center;
    }
    .calendar-table {
        margin-top: 22px;
        width: 100%;
        text-align: center;
        .week-day {
            height: 17px;
            font-size:10px;
            line-height:17px;
            color: #8D92A3;
        }
        .tr {
            display: flex;
            align-items: center;
        }
        .td {
            flex: 1;
        }
        .t-body {
            .td {
                position: relative;
                height: 38px;
                .date-info {
                    display: flex;
                    flex-direction: column;
                    align-items: center;
                    justify-content: center;
                    position: absolute;
                    top: 0;
                    left: 0;
                    right: 0;
                    bottom: 0;
                    margin: auto;
                    width: 28px;
                    height: 28px;
                    border-radius: 50%;
                    font-size: 11px;
                    &.now {
                        background-color: @backGreyColor;
                    }
                    &.select {
                        background-color: @focusColor;
                        color: @backFontColor;
                    }
                }
                &.out-month {
                    visibility: hidden;
                }
            }
        }
        .area {
            transition: height .3s;
            overflow: hidden;
        }
        .month {
            position: relative;
            &.animation {
                transition: transform .3s;
            }
            .prev,
            .next {
                position: absolute;
                left: 0;
                right: 0;
            }
            .prev {
                top: 0;
                transform: translateY(-100%);
            }
            .next {
                bottom: 0;
                transform: translateY(100%);
            }
        }
    }
</style>
