<script setup lang="ts">
import { reactive, onMounted, watch } from "vue";
import { storeToRefs } from "pinia";

// import { useStoreAttendanceManagement } from "@/stores/attendanceManagementData";

// const storeAttendanceManagement = useStoreAttendanceManagement();
// const { attendanceManagement } = storeToRefs(storeAttendanceManagement);

import { getCalendar } from "@/libs/calendar";
import { caluCurrentMonthly, getDayInUTC } from "@/libs/date";
import { getWindowSize } from "@/libs/windowSize";

const { attendanceRecords } = defineProps(["attendanceRecords"]);

watch(attendanceRecords, () => {
  calcCumulativeTotal();
});

onMounted(() => {
  init();
});

const init = () => {
  getCalendar(reactiveData);

  getWindowSize(reactiveData);
  window.addEventListener("resize", getWindowSize);
};

const reactiveData = reactive({
  // attendanceTypes: ["残業", "深夜", "休出", "年休", "欠勤", "公休", "休業"],
  attendanceTypes: {
    overTime: "残業",
    lateNightOverTime: "深夜",
    holidayWorkTime: "休出",
    annualVacation: "年休",
    absenteeism: "欠勤",
    publicHolidayTime: "公休",
    companyClosureTime: "休業",
  },

  calendars: {},
  cumulativeTotal: {
    overTime: 0.0,
    lateNightOverTime: 0.0,
    holidayWorkTime: 0.0,
    annualVacation: 0.0,
    absenteeism: 0.0,
    publicHolidayTime: 0.0,
    companyClosureTime: 0.0,
  },
  start_month: caluCurrentMonthly(),
  end_month: caluCurrentMonthly(),
  inner_width: 0,
  inner_height: 0,
});

const calcCumulativeTotal = () => {
  for (let i = 0; i < attendanceRecords.length; i++) {
    reactiveData.cumulativeTotal.overTime += attendanceRecords[i].overTime;

    reactiveData.cumulativeTotal.lateNightOverTime +=
      attendanceRecords[i].lateNightOverTime;

    reactiveData.cumulativeTotal.holidayWorkTime +=
      attendanceRecords[i].holidayWorkTime;

    reactiveData.cumulativeTotal.absenteeism +=
      attendanceRecords[i].absenteeism;

    reactiveData.cumulativeTotal.publicHolidayTime +=
      attendanceRecords[i].publicHolidayTime;
  }
};
</script>

<template>
  <div class="flex">
    <!-- 出勤メモ -->
    <div class="w-60">
      <div
        class="flex flex-col justify-center items-center h-24 bg-green-600 font-bold text-sm text-white"
      >
        出勤時間に関するメモ
      </div>
      <div>
        <input type="text" class="w-full border h-56" />
      </div>
    </div>
    <!-- 出勤メモここまで -->

    <!-- 種別 -->
    <table class="flex flex-col items-center border-l border-r">
      <thead>
        <tr>
          <th
            class="flex flex-col justify-center h-24 w-20 bg-green-600 font-bold text-sm text-white"
          >
            種別
          </th>
        </tr>
      </thead>

      <tbody class="w-full h-full">
        <tr class="flex flex-col items-center w-full">
          <td
            class="flex justify-center items-center w-full font-bold h-8 text-sm border-b"
          >
            {{ reactiveData.attendanceTypes.overTime }}
          </td>
          <td
            class="flex justify-center items-center w-full font-bold h-8 text-sm border-b"
          >
            {{ reactiveData.attendanceTypes.lateNightOverTime }}
          </td>
          <td
            class="flex justify-center items-center w-full font-bold h-8 text-sm border-b"
          >
            {{ reactiveData.attendanceTypes.holidayWorkTime }}
          </td>
          <td
            class="flex justify-center items-center w-full font-bold h-8 text-sm border-b"
          >
            {{ reactiveData.attendanceTypes.annualVacation }}
          </td>
          <td
            class="flex justify-center items-center w-full font-bold h-8 text-sm border-b"
          >
            {{ reactiveData.attendanceTypes.absenteeism }}
          </td>
          <td
            class="flex justify-center items-center w-full font-bold h-8 text-sm border-b"
          >
            {{ reactiveData.attendanceTypes.publicHolidayTime }}
          </td>
          <td
            class="flex justify-center items-center w-full font-bold h-8 text-sm border-b"
          >
            {{ reactiveData.attendanceTypes.companyClosureTime }}
          </td>
        </tr>
      </tbody>
    </table>
    <!-- 種別ここまで -->

    <!-- 出勤カレンダー -->
    <table class="flex border-t border-b border-r">
      <div
        v-for="(dayObj, index) in reactiveData.calendars.days"
        v-bind:key="index"
      >
        <thead class="border-r border-b">
          <tr>
            <th class="flex flex-col justify-center h-24 w-8 font-bold">
              <span
                v-bind:class="
                  dayObj.dayOfWeek === '土' || dayObj.dayOfWeek === '日'
                    ? 'text-red-800'
                    : ''
                "
                >{{ dayObj.day }}</span
              >
              <span
                v-bind:class="
                  dayObj.dayOfWeek === '土' || dayObj.dayOfWeek === '日'
                    ? 'text-red-800'
                    : ''
                "
                >{{ dayObj.dayOfWeek }}</span
              >
            </th>
          </tr>
        </thead>

        <div
          v-for="(record, index) in attendanceRecords"
          v-bind:key="index"
          class="flex flex-col items-center border-r"
        >
          <tbody
            v-if="dayObj.day === getDayInUTC(record.workDate)"
            class="w-full flex flex-col items-center border-r"
          >
            <tr class="flex flex-col items-center h-36 w-full">
              <td
                class="flex justify-center items-center w-full text-base h-8 border-b"
              >
                {{ record.overTime > 0.0 ? record.overTime.toFixed(1) : "0.0" }}
              </td>
              <td
                class="flex justify-center items-center w-full text-base h-8 border-b"
              >
                {{
                  record.lateNightOverTime > 0.0
                    ? record.lateNightOverTime.toFixed(1)
                    : ""
                }}
              </td>
              <td
                class="flex justify-center items-center w-full text-base h-8 border-b"
              >
                {{
                  record.holidayWorkTime > 0.0
                    ? record.holidayWorkTime.toFixed(1)
                    : ""
                }}
              </td>
              <td
                class="flex justify-center items-center w-full text-base h-8 border-b"
              >
                {{
                  record.anuualVacation > 0.0
                    ? record.anuualVacation.toFixed(1)
                    : ""
                }}
              </td>
              <td
                class="flex justify-center items-center w-full text-base h-8 border-b"
              >
                {{
                  record.absenteeism > 0.0 ? record.absenteeism.toFixed(1) : ""
                }}
              </td>
              <td class="flex justify-center items-center w-full text-base h-8">
                {{
                  record.publicHolidayTime > 0.0
                    ? record.publicHolidayTime.toFixed(1)
                    : ""
                }}
              </td>
            </tr>
          </tbody>
        </div>
      </div>
    </table>

    <!-- 出勤カレンダーここまで -->

    <!-- 累計 -->
    <table class="flex border-r border-l">
      <div>
        <thead>
          <tr>
            <th
              class="flex flex-col justify-center h-24 w-24 bg-green-600 font-bold text-sm text-white"
            >
              <span>累計</span>
            </th>
          </tr>
        </thead>
        <tbody class="w-full h-full">
          <tr class="flex flex-col items-center w-full">
            <td
              class="flex justify-center items-center w-full text-base h-8 border-b"
            >
              {{ reactiveData.cumulativeTotal.overTime.toFixed(1) }}&nbsp;時間
            </td>
            <td
              class="flex justify-center items-center w-full text-base h-8 border-b"
            >
              {{
                reactiveData.cumulativeTotal.lateNightOverTime.toFixed(1)
              }}&nbsp;時間
            </td>
            <td
              class="flex justify-center items-center w-full text-base h-8 border-b"
            >
              {{
                reactiveData.cumulativeTotal.holidayWorkTime.toFixed(1)
              }}&nbsp;時間
            </td>
            <td
              class="flex justify-center items-center w-full text-base h-8 border-b"
            >
              {{
                reactiveData.cumulativeTotal.annualVacation.toFixed(1)
              }}&nbsp;日
            </td>
            <td
              class="flex justify-center items-center w-full text-base h-8 border-b"
            >
              {{ reactiveData.cumulativeTotal.absenteeism.toFixed(1) }}&nbsp;日
            </td>
            <td
              class="flex justify-center items-center w-full text-base h-8 border-b"
            >
              {{
                reactiveData.cumulativeTotal.publicHolidayTime.toFixed(1)
              }}&nbsp;日
            </td>
            <td
              class="flex justify-center items-center w-full text-base h-8 border-b"
            >
              {{
                reactiveData.cumulativeTotal.companyClosureTime.toFixed(1)
              }}&nbsp;日
            </td>
          </tr>
        </tbody>
      </div>
    </table>

    <!-- 累計ここまで -->
  </div>
</template>

<style scoped></style>
