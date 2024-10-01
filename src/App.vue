<script setup>
import { ref } from 'vue'
import axios from 'axios'
const locationName = ref('臺中市')
const locationNameTemp = ref('')
const isVisible = ref(false)
const isNotFind = ref(false)
const imgs = ref(['wx3.jpg', 'wx1.jpg', 'wx2.jpg'])
const weatherBg = ref({ before: '', after: '' })

const resData = ref({
  datasetDescription: '',
  locationName: '',
  data: []
})

const apiBase = 'https://opendata.cwa.gov.tw/api'

const getWrather = () => {
  const searchLocationNameArray = ref(locationName.value.split(''))
  if (searchLocationNameArray.value[0] == '台') searchLocationNameArray.value[0] = '臺'

  weatherBg.value.before = weatherBg.value.after
  isVisible.value = false
  setTimeout(async () => {
    try {
      const res = await axios.get(
        `${apiBase}/v1/rest/datastore/F-C0032-001?Authorization=CWA-9462CDFF-C615-4751-8067-E909546691AD&limit=5&locationName=${searchLocationNameArray.value.join(
          ''
        )}`
      )
      const data = res.data.records

      if (data.location.length) {
        resData.value.datasetDescription = data.datasetDescription
        resData.value.locationName = data.location[0].locationName

        let weatherElement = data.location[0].weatherElement
        const array = weatherElement.shift()

        resData.value.data = array.time.reduce((a, timeObj) => {
          a.push({
            startTime: timeObj.startTime,
            endTime: timeObj.endTime,
            [array.elementName]: timeObj.parameter.parameterName
          })
          return a
        }, [])

        resData.value.data = weatherElement.reduce((item, obj) => {
          item.forEach((element, index) => {
            const idx = obj.time.findIndex(
              (i) => i.startTime === element.startTime && i.endTime === element.endTime
            )
            Object.assign(item[index], {
              [obj.elementName]: obj.time[idx].parameter.parameterName
            })
          })
          return item
        }, resData.value.data)

        // 選擇背景
        const weatherWx = resData.value.data[0].Wx.slice()
        const weatherSwitch = ['晴', '雨']
        weatherBg.value.after = ''
        weatherSwitch.forEach((item, key) => {
          if (weatherWx.includes(item)) return (weatherBg.value.after = imgs.value[key])
        })
        console.log(weatherBg.value.after, !weatherBg.value.after)
        if (!weatherBg.value.after) weatherBg.value.after = imgs.value[2]

        isVisible.value = true
        isNotFind.value = false
      } else {
        locationNameTemp.value = locationName.value
        isVisible.value = true
        isNotFind.value = true
      }
    } catch (err) {
      console.log(err)
    }
  }, 400)
}

getWrather()
</script>

<template>
  <main :style="{ backgroundImage: `url(/src/assets/images/${weatherBg.before})` }">
    <div
      class="main-bg"
      :style="{ backgroundImage: `url(/src/assets/images/${weatherBg.after})` }"
      :class="{ show: isVisible }"
    ></div>
    <article class="container-fluid position-relative">
      <div class="row vh-100 align-items-center justify-content-center">
        <div class="col-md-11 col-lg-8 col-xl-6">
          <input
            type="text"
            placeholder="輸入地區"
            class="form-control rounded-pill p-3 mb-3"
            v-model="locationName"
            @keypress.enter="getWrather"
          />
          <div id="weatherBox" :class="{ show: isVisible }">
            <h1 v-if="isNotFind" class="p-5 text-center">
              未找到 "{{ locationNameTemp }}" 相關資料
            </h1>
            <div
              class="d-flex flex-column flex-md-row border rounded-pill overflow-hidden"
              style="min-height: 400px"
              v-if="!isNotFind"
            >
              <div id="weatherMain" class="col-md-8 py-4 ps-5">
                <div class="d-flex flex-column h-100">
                  <div class="mb-auto">
                    <p style="font-size: 12px">/ 三十六小時天氣預報</p>
                    <h3>{{ resData.locationName }}</h3>
                  </div>
                  <div>
                    <p style="font-size: 24px">{{ resData.data[0].MinT }}~</p>
                    <p class="lh-1">
                      <span class="maxT">{{ resData.data[0].MaxT }}</span>
                    </p>
                    <p>{{ resData.data[0].Wx }}</p>
                    <p style="font-size: 8px">{{ resData.data[0].startTime }}</p>
                  </div>
                </div>
              </div>
              <div class="col-md-4 text-dark bg-light p-2 py-4 py-ms-2">
                <ul id="weatherList" class="d-flex flex-md-column h-100 text-center">
                  <template v-for="(val, key) of resData.data" :key="key">
                    <li class="flex-fill align-content-center" v-if="key">
                      <p style="font-size: 8px">{{ val.startTime }}</p>
                      <p class="lh-1">
                        <span>{{ val.MinT }}~ </span>
                        <span class="maxT" style="font-size: 60px">{{ val.MaxT }}</span>
                      </p>
                      <p class="m-0" style="font-size: 12px">{{ val.Wx }}</p>
                    </li>
                  </template>
                </ul>
              </div>
            </div>
          </div>
        </div>
      </div>
    </article>
  </main>
</template>

<style>
.rounded-pill {
  border-radius: 20px !important;
}

main,
.main-bg {
  background: center;
  background-size: cover;
  overflow: auto;
}
.main-bg {
  position: absolute;
  width: 100%;
  height: 100%;
  opacity: 0;
  transition: opacity 0.5s;
}
.main-bg.show {
  opacity: 1;
}

#weatherBox {
  position: relative;
  color: #fff;
  transform: translate(0, 20px);
  opacity: 0;
  transition: all 0.3s;
}
#weatherBox.show {
  transform: translate(0, 0);
  opacity: 1;
}
#weatherBox::before {
  content: '';
  position: absolute;
  display: block;
  width: 100%;
  height: 100%;
  border-radius: 20px;
  box-shadow: 0 0 50px rgba(255, 255, 255, 1) inset;
}
#weatherBox::after {
  content: 'WEATHER FORECAST';
  position: absolute;
  text-align: center;
  font-size: 0.8rem;
  right: 0;
  bottom: -30px;
}

#weatherMain {
  background-color: rgba(50, 50, 50, 0.8);
}

#weatherList {
  margin: 0;
  padding: 0;
}
#weatherList li {
  list-style: none;
}
#weatherList li:nth-child(n + 2) {
  border-left: 1px solid;
}
/* // md devices  */
@media (min-width: 768px) {
  #weatherBox::after {
    right: -6rem;
    bottom: 50%;
    transform: rotate(90deg);
  }
  #weatherList li:nth-child(n + 2) {
    border-left: none;
    border-top: 1px solid;
  }
}

.maxT {
  position: relative;
  font-size: 80px;
  font-weight: bolder;
}
.maxT::after {
  content: '℃';
  font-size: 32px;
  position: absolute;
  top: 0;
  right: -40px;
}
</style>
