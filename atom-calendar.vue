<script setup lang="ts">
import { ref, onMounted } from "vue";
import calendar from "./calendar";

interface Props {
  defaultStart?: number; // 星期开始时间  默认为0 ---> 星期天
  headerLineColor?: string; // 头部线颜色
  lastMonthIcon?: string; // 头部上月图标
  nextMonthIcon?: string; // 头部下月图标
  lastYearIcon?: string; // 头部上一年图标
  nextYearIcon?: string; // 头部下一年图标
  currentDayBgColor?: string; // 当前日期背景色颜色
  currentDayColor?: string; // 当前日期文字颜色
  festivalColor?: string; //重要节日颜色
  clickItemColor?: string; // 点击日期背景颜色
  viewType?: string; // 视图类型  month/week
  showHeader: boolean; // 是否显示头部
}

interface Emits {
  (event: "handleCurrent", item: any): void; // 点击日期项事件
  (event: "handleChange", item: string): void; // 日历月份切换
}

interface IMonthDateTree {
  days?: number | string;
  month: number | string;
  year: number | string;
  lunarDate: any;
  type: string;
  isCurrenMonth?: boolean;
  currentColor: string;
}

interface ICurrentWeekTree extends IMonthDateTree {}

const props = defineProps<Props>();

const emits = defineEmits<Emits>();

// 头部区域样式
const headerStyle = ref({
  "--header-border_b": props.headerLineColor ?? "#008080", // header bottom color
});

// 上月图标
const lastMonthIcon = ref(props.lastMonthIcon);

// 下月图标
const nextMonthIcon = ref(props.nextMonthIcon);

// 上一年图标
const lastYearIcon = ref(props.lastYearIcon);

// 下一年图标
const nextYearIcon = ref(props.nextYearIcon);

// 星期字典
const weekDict = ["日", "一", "二", "三", "四", "五", "六"];

// 默认星期开始
const defaultStart = ref(props.defaultStart ?? 0);

// 是否显示头部
const showHeader = ref(props.showHeader ?? true);

// 星期
const weekDay = () => {
  return weekDict
    .slice(defaultStart.value)
    .concat(weekDict.slice(0, defaultStart.value));
};

// 当前年
const currentYear = ref(new Date().getFullYear());

// 当前月
const currentMonth = ref(new Date().getMonth() + 1);

// 当前日
const currentDay = ref(new Date().getDate());

// 周视图或月视图
const viewType = ref(props.viewType ?? "month");

// 当前月日期树
const currentMonthDateTree = ref<any[]>([]);

// 当前日期树样式
const currentMonthDateTreeStyle = ref({
  "--currentColor": props.currentDayColor ?? "#000",
  "--currentBgColor": props.currentDayBgColor ?? "#008080",
  "--festivalColor": props.festivalColor ?? "#008080",
});

// 获取当前月份数据
const getCurrentMonthDateTree = (y: number, m: number) => {
  let monthDateTree: IMonthDateTree[] = [];

  // 当月第一天
  const firstDay = new Date(y, m - 1, 1).getDay();

  // 当月最后一天
  const lastDay = new Date(y, m, 0).getDate();

  // 上月的最后一天
  const lastDayOfLastMonth = new Date(y, m - 1, 0).getDate();

  // 当月天数
  const days = new Date(y, m, 0).getDate();

  // 默认星期开始
  let weekStart = defaultStart.value === 7 ? 0 : defaultStart.value;

  //月初有几天是上月的
  const prevMonthDays = (() => {
    // 周初有几天是上个月的
    if (firstDay == weekStart) {
      return 0;
    } else if (firstDay > weekStart) {
      return firstDay - weekStart;
    } else {
      return 7 - weekStart + firstDay;
    }
  })();

  // 结束还有几天是下个月的
  const nextMonthDays = 7 - ((prevMonthDays + lastDay) % 7);

  /**
   * 拼接当月日记树格子
   * 1. 拼接上个月日期格子（如果有的话）
   * 2. 拼接当月
   * 3. 拼接下个月日期格子（如果有的话）
   */

  //  上月年份
  const lastY = new Date(y, m - 1, 0).getFullYear();

  // 下月年份
  const nextY = new Date(y, m + 1, 0).getFullYear();

  // 上月月份
  const lastM = new Date(y, m - 1, 0).getMonth() + 1;

  // 下月月份
  const nextM = new Date(y, m + 1, 0).getMonth() + 1;

  // 拼接上个月日期格子
  if (prevMonthDays > 0) {
    for (let i = 1; i <= prevMonthDays; i++) {
      monthDateTree.push({
        days: formatNum(lastDayOfLastMonth - prevMonthDays + i),
        month: lastM,
        year: lastY,
        lunarDate: translateLunar(
          lastY,
          lastM,
          formatNum(lastDayOfLastMonth - prevMonthDays + i)
        ),
        type: "prev-month",
        currentColor: "",
      });
    }
  }

  //拼接当月
  for (let i = 1; i <= days; i++) {
    monthDateTree.push({
      days: formatNum(i),
      month: formatNum(m),
      year: y,
      lunarDate: translateLunar(y, formatNum(m), formatNum(i)),
      type: "current-month",
      isCurrenMonth: true, // 是否是当前月
      currentColor: "",
    });
  }

  // 拼接下个月日期格子
  if (nextMonthDays > 0) {
    for (let i = 0; i < nextMonthDays; i++) {
      monthDateTree.push({
        days: formatNum(i + 1),
        month: nextM,
        year: nextY,
        lunarDate: translateLunar(nextY, nextM, formatNum(i + 1)),
        type: "next-month",
        currentColor: "",
      });
    }
  }

  console.log(monthDateTree);

  return monthDateTree;
};

// 根据当天所在星期数获取本周所有日期
const getCurrentWeekDateTree = (y: number, m: number, d: number) => {
  let currentWeekTree: ICurrentWeekTree[] = [];

  // 默认星期开始
  let weekStart = defaultStart.value === 7 ? 0 : defaultStart.value;

  // 获取当前日期所在星期数
  let currentWeek = new Date(y, m - 1, d).getDay();

  console.log(y, m, d, currentWeek);

  // 当前星期数前面有几天
  let prevDays = weekStart - currentWeek;
  if (prevDays <= 0) {
    prevDays = Math.abs(prevDays);
  }

  // 当前星期数后面有几天
  let nextDays = 7 - (currentWeek + 1);
  if (nextDays >= 7) {
    nextDays = 0;
  }

  // 拼接当前星期前几天
  for (let i = 0; i < prevDays; i++) {
    currentWeekTree.push({
      days: formatNum(d - currentWeek + i),
      month: formatNum(m),
      year: y,
      lunarDate: translateLunar(
        y,
        formatNum(m),
        formatNum(d - currentWeek + i)
      ),
      type: "prev-week",
      isCurrenMonth: true,
      currentColor: "",
    });
  }

  // 拼接当天
  currentWeekTree.push({
    days: formatNum(d),
    month: formatNum(m),
    year: y,
    lunarDate: translateLunar(y, m, d),
    type: "current-week",
    isCurrenMonth: true,
    currentColor: "",
  });

  // 拼接当前星期后几天
  for (let i = 0; i < nextDays; i++) {
    currentWeekTree.push({
      days: formatNum(d + i + 1),
      month: formatNum(m),
      year: y,
      lunarDate: translateLunar(y, formatNum(m), formatNum(d + i + 1)),
      type: "next-week",
      isCurrenMonth: true,
      currentColor: "",
    });
  }

  return currentWeekTree;
};

// 公历日期转农历日期
const translateLunar = (y: any, m: any, d: any) => {
  const lunarDate = calendar.solar2lunar(y, m, d);

  // 阳历日期
  const data = `${formatNum(m)}-${formatNum(d)}`;
  // 农历日期 lMonth  lDay
  const lunar = `${lunarDate.lMonth}-${lunarDate.lDay}`;

  return {
    ...lunarDate,
    festival: calendar.getFestival(
      data,
      lunar,
      lunarDate.cMonth,
      lunarDate.nWeek,
      lunarDate.ncWeek
    ),
  };
};

// 点击日期项
const handleCurrentDays = (item: any) => {
  if (!item.isCurrenMonth) return;
  // 切换点击颜色
  currentMonthDateTree.value.forEach((i) => {
    i.currentColor =
      i.days === item.days && i.type === item.type
        ? props.clickItemColor ?? "#00808182"
        : "";
  });
  emits("handleCurrent", { ...item });
};

// 点击下一月
const handleNextMonth = () => {
  if (currentMonth.value === 12) {
    currentYear.value++;
    currentMonth.value = 1;
  } else {
    currentMonth.value++;
  }
  init();
};

// 点击上一月
const handlePrevMonth = () => {
  if (currentMonth.value === 1) {
    currentYear.value--;
    currentMonth.value = 12;
  } else {
    currentMonth.value--;
  }
  init();
};

// 点击上一年
const handlePrevYear = () => {
  currentYear.value--;
  init();
};

// 点击下一年
const handleNextYear = () => {
  currentYear.value++;
  init();
};

// 切换视图
const handleViewChange = () => {
  viewType.value = viewType.value === "month" ? "week" : "month";
  void init();
};

// 初始化
const init = () => {
  // 如果不是当前日，则重置为每月一号
  if (viewType.value === "week") {
    let y = new Date().getFullYear();
    let m = new Date().getMonth() + 1;
    if (y !== currentYear.value || m !== currentMonth.value) {
      currentMonthDateTree.value = getCurrentMonthDateTree(
        currentYear.value,
        currentMonth.value
      ).slice(0, 7);
    } else {
      currentMonthDateTree.value = getCurrentWeekDateTree(
        currentYear.value,
        currentMonth.value,
        currentDay.value
      );
    }
  } else if (viewType.value === "month") {
    currentMonthDateTree.value = getCurrentMonthDateTree(
      currentYear.value,
      currentMonth.value
    );
  }
};

// 补零
const formatNum = (num: any) => {
  let res = Number(num);
  return res < 10 ? "0" + res : res;
};

// 轮播值
const currentSwiperItem = ref(0);

// 轮播图change事件
const handleChange = (e: { detail: { current: number } }) => {
  let current = e.detail.current;
  if (
    current - currentSwiperItem.value == 1 ||
    current - currentSwiperItem.value == -2
  ) {
    handleNextMonth();
  } else {
    handlePrevMonth();
  }
  currentSwiperItem.value = current;
  void handleCustomChange();
};

// 自定义change事件
const handleCustomChange = () => {
  const currentYearMonth = `${currentYear.value}年${formatNum(
    currentMonth.value
  )}月`;
  emits("handleChange", currentYearMonth);
};

onMounted(() => {
  void handleCustomChange();
  void init();
});
</script>
<template>
  <view class="content w-full h-auto">
    <!-- header -->
    <view
      class="header w-full h-[90rpx] flex justify-center items-center"
      :style="headerStyle"
      v-if="showHeader"
    >
      <!-- 上一年箭头 -->
      <view
        class="last w-[40rpx] h-[40rpx] mr-[40rpx] overflow-hidden"
        @click="handlePrevYear"
      >
        <image
          :src="lastYearIcon ?? '/static/iconfont/icon_chevrons_left.png'"
          class="w-full h-full"
          mode=""
        >
        </image>
      </view>
      <!-- 上个月箭头 -->
      <view
        class="last w-[34rpx] h-[34rpx] overflow-hidden"
        @click="handlePrevMonth"
      >
        <image
          :src="lastMonthIcon ?? '/static/iconfont/icon_arrow_left.png'"
          class="w-full h-full"
          mode=""
        >
        </image>
      </view>
      <!-- 显示区 -->
      <view class="look mx-[50rpx] text-[20px] font-bold">
        {{ currentYear }}年{{ formatNum(currentMonth) }}月
      </view>
      <!-- 下个月箭头 -->
      <view
        class="next w-[34rpx] h-[34rpx] overflow-hidden"
        @click="handleNextMonth"
      >
        <image
          :src="nextMonthIcon ?? '/static/iconfont/icon_arrow_right.png'"
          class="w-full h-full"
          mode=""
        >
        </image>
      </view>
      <!-- 下一年箭头 -->
      <view
        class="next w-[40rpx] h-[40rpx] ml-[40rpx] overflow-hidden"
        @click="handleNextYear"
      >
        <image
          :src="nextYearIcon ?? '/static/iconfont/icon_chevrons_right.png'"
          class="w-full h-full"
          mode=""
        >
        </image>
      </view>
    </view>

    <!-- week -->
    <view class="week w-full h-[90rpx] flex items-center">
      <view
        class="week-day flex-1 text-center"
        v-for="(item, index) in weekDay()"
        :key="index"
        >{{ item }}</view
      >
    </view>

    <!-- days data -->
    <view
      class="swiper w-full overflow-hidden duration-1000 transition-all delay-400"
      :style="{
        height: viewType === 'month' ? '450rpx' : '100rpx',
      }"
    >
      <swiper class="w-full h-full" circular @change="handleChange">
        <swiper-item>
          <view class="w-full flex items-center flex-wrap">
            <view
              :style="currentMonthDateTreeStyle"
              class="days-item w-[calc(100%/7)] h-[90rpx] flex justify-center items-center"
              v-for="(i, idx) in currentMonthDateTree"
              :key="idx"
            >
              <view
                @click="handleCurrentDays(i)"
                :style="{
                  backgroundColor:
                    formatNum(currentDay) == i.days &&
                    new Date().getFullYear() == i.year &&
                    formatNum(new Date().getMonth() + 1) == i.month
                      ? 'var(--currentBgColor)'
                      : i.currentColor,
                  color:
                    i.lunarDate.Term !== null || i.lunarDate.festival !== null
                      ? formatNum(currentDay) == i.days &&
                        new Date().getFullYear() == i.year &&
                        formatNum(new Date().getMonth() + 1) == i.month
                        ? '#000'
                        : 'var(--festivalColor)'
                      : 'var(--currentColor)',
                }"
                class="w-[90rpx] h-[90rpx] rounded-full flex justify-center items-center flex-col"
              >
                <view
                  class="text-center font-normal text-[16px]"
                  v-if="i.isCurrenMonth"
                >
                  {{ i.days }}
                </view>

                <view
                  v-if="i.isCurrenMonth"
                  class="w-[80%] text-center font-normal text-[12px] truncate"
                >
                  {{
                    i.lunarDate.festival !== null
                      ? i.lunarDate.festival
                      : i.lunarDate.Term !== null
                      ? i.lunarDate.Term
                      : i.lunarDate.IDayCn === "初一"
                      ? i.lunarDate.IMonthCn
                      : i.lunarDate.IDayCn
                  }}
                </view>
              </view>
            </view>
          </view>
        </swiper-item>
        <swiper-item>
          <view class="w-full flex items-center flex-wrap">
            <view
              :style="currentMonthDateTreeStyle"
              class="days-item w-[calc(100%/7)] h-[90rpx] flex justify-center items-center"
              v-for="(i, idx) in currentMonthDateTree"
              :key="idx"
            >
              <view
                @click="handleCurrentDays(i)"
                :style="{
                  backgroundColor:
                    formatNum(currentDay) == i.days &&
                    new Date().getFullYear() == i.year &&
                    formatNum(new Date().getMonth() + 1) == i.month
                      ? 'var(--currentBgColor)'
                      : i.currentColor,
                  color:
                    i.lunarDate.Term !== null || i.lunarDate.festival !== null
                      ? formatNum(currentDay) == i.days &&
                        new Date().getFullYear() == i.year &&
                        formatNum(new Date().getMonth() + 1) == i.month
                        ? '#000'
                        : 'var(--festivalColor)'
                      : 'var(--currentColor)',
                }"
                class="w-[90rpx] h-[90rpx] rounded-full flex justify-center items-center flex-col"
              >
                <view
                  class="text-center font-normal text-[16px]"
                  v-if="i.isCurrenMonth"
                >
                  {{ i.days }}
                </view>

                <view
                  v-if="i.isCurrenMonth"
                  class="w-[80%] text-center font-normal text-[12px] truncate"
                >
                  {{
                    i.lunarDate.festival !== null
                      ? i.lunarDate.festival
                      : i.lunarDate.Term !== null
                      ? i.lunarDate.Term
                      : i.lunarDate.IDayCn === "初一"
                      ? i.lunarDate.IMonthCn
                      : i.lunarDate.IDayCn
                  }}
                </view>
              </view>
            </view>
          </view>
        </swiper-item>
        <swiper-item>
          <view class="w-full flex items-center flex-wrap">
            <view
              :style="currentMonthDateTreeStyle"
              class="days-item w-[calc(100%/7)] h-[90rpx] flex justify-center items-center"
              v-for="(i, idx) in currentMonthDateTree"
              :key="idx"
            >
              <view
                @click="handleCurrentDays(i)"
                :style="{
                  backgroundColor:
                    formatNum(currentDay) == i.days &&
                    new Date().getFullYear() == i.year &&
                    formatNum(new Date().getMonth() + 1) == i.month
                      ? 'var(--currentBgColor)'
                      : i.currentColor,
                  color:
                    i.lunarDate.Term !== null || i.lunarDate.festival !== null
                      ? formatNum(currentDay) == i.days &&
                        new Date().getFullYear() == i.year &&
                        formatNum(new Date().getMonth() + 1) == i.month
                        ? '#000'
                        : 'var(--festivalColor)'
                      : 'var(--currentColor)',
                }"
                class="w-[90rpx] h-[90rpx] rounded-full flex justify-center items-center flex-col"
              >
                <view
                  class="text-center font-normal text-[16px]"
                  v-if="i.isCurrenMonth"
                >
                  {{ i.days }}
                </view>

                <view
                  v-if="i.isCurrenMonth"
                  class="w-[80%] text-center font-normal text-[12px] truncate"
                >
                  {{
                    i.lunarDate.festival !== null
                      ? i.lunarDate.festival
                      : i.lunarDate.Term !== null
                      ? i.lunarDate.Term
                      : i.lunarDate.IDayCn === "初一"
                      ? i.lunarDate.IMonthCn
                      : i.lunarDate.IDayCn
                  }}
                </view>
              </view>
            </view>
          </view>
        </swiper-item>
      </swiper>
    </view>

    <!-- 收缩栏 -->
    <view class="header w-[100%] h-[90rpx] flex justify-center items-center">
      <!-- 月视图 -->
      <view
        class="w-[60rpx] h-[10rpx]"
        v-show="viewType === 'month'"
        @click="handleViewChange"
      >
        <image
          class="w-full h-full"
          src="/static/iconfont/line.png"
          mode="scaleToFill"
        />
      </view>

      <!-- 周视图 -->
      <view
        class="w-[50rpx] h-[50rpx]"
        v-show="viewType === 'week'"
        @click="handleViewChange"
      >
        <image
          class="w-full h-full"
          src="/static/iconfont/icon_arrow_down.png"
          mode="scaleToFill"
        />
      </view>
    </view>
  </view>
</template>

<style lang="scss">
.content {
  .header {
    border-bottom: 1rpx solid var(--header-border_b);
  }
}
</style>
