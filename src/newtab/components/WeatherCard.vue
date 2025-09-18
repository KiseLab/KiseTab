<template>
  <div class="weather-card" role="region" aria-label="当前天气">
    <template v-if="loading">
      <div class="msg">加载中…</div>
    </template>

    <template v-else-if="disabled">
      <div class="msg">天气服务已禁用或无法获取地理位置</div>
    </template>

    <template v-else-if="error">
      <div class="msg">{{ error }}</div>
    </template>

    <template v-else>
      <div class="top">
        <div class="location">{{ cityDisplay }}</div>
        <div class="cond">{{ condition }}</div>
      </div>
      <div class="bottom">
        <img v-if="iconUrl" :src="iconUrl" alt="天气图标" class="icon" />
        <div class="temp">{{ Math.round(temp) }}<span>°C</span></div>
      </div>
    </template>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from "vue";

const loading = ref(true);
const disabled = ref(false);
const error = ref<string | null>(null);

const city = ref("");
const temp = ref(28);
const condition = ref("");
const icon = ref<string | null>(null);

const iconUrl = ref<string | null>(null);

let abortCtrl: AbortController | null = null;

const API_KEY = import.meta.env.VITE_WEATHER_API_KEY || "";

// 国家映射
const COUNTRY_MAP: Record<string, string> = {
  CN: "中国",
};

// 省份映射
const PROVINCE_MAP: Record<string, string> = {
  Beijing: "北京市",
  Tianjin: "天津市",
  Hebei: "河北省",
  Shanxi: "山西省",
  InnerMongolia: "内蒙古自治区",
  Liaoning: "辽宁省",
  Jilin: "吉林省",
  Heilongjiang: "黑龙江省",
  Shanghai: "上海市",
  Jiangsu: "江苏省",
  Zhejiang: "浙江省",
  Anhui: "安徽省",
  Fujian: "福建省",
  Jiangxi: "江西省",
  Shandong: "山东省",
  Henan: "河南省",
  Hubei: "湖北省",
  Hunan: "湖南省",
  Guangdong: "广东省",
  Guangxi: "广西壮族自治区",
  Hainan: "海南省",
  Chongqing: "重庆市",
  Sichuan: "四川省",
  Guizhou: "贵州省",
  Yunnan: "云南省",
  Tibet: "西藏自治区",
  Shaanxi: "陕西省",
  Gansu: "甘肃省",
  Qinghai: "青海省",
  Ningxia: "宁夏回族自治区",
  Xinjiang: "新疆维吾尔自治区",
  Taiwan: "台湾省",
  HongKong: "香港特别行政区",
  Macao: "澳门特别行政区",
};

function makeIconUrl(code: string) {
  return `https://openweathermap.org/img/wn/${code}@2x.png`;
}

async function fetchWeather(lat: number, lon: number) {
  if (!API_KEY) {
    error.value = "未配置天气 API Key (VITE_WEATHER_API_KEY)";
    loading.value = false;
    return;
  }

  abortCtrl = new AbortController();
  const signal = abortCtrl.signal;

  try {
    const onecallUrl = `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${API_KEY}&lang=zh_cn`;
    const res = await fetch(onecallUrl, { signal });
    if (!res.ok) throw new Error(`天气 API 返回 ${res.status}`);
    const data = await res.json();

    const toCelsius = (v: number) => {
      // 如果温度看起来像开尔文（>100），则转换为摄氏度；否则视为已是摄氏度
      return typeof v === "number" ? (v > 100 ? v - 273.15 : v) : v;
    };

    if (data && data.current) {
      const cur = data.current;
      temp.value = toCelsius(cur.temp);
      const w = Array.isArray(cur.weather) && cur.weather[0];
      condition.value = (w && (w.description || w.main)) || "暂无信息";
      icon.value = w && w.icon ? w.icon : null;
      iconUrl.value = icon.value ? makeIconUrl(icon.value) : null;
    } else if (data && data.main) {
      // 当前天气接口返回结构
      temp.value = toCelsius(data.main.temp);
      const w = Array.isArray(data.weather) && data.weather[0];
      condition.value = (w && (w.description || w.main)) || "暂无信息";
      icon.value = w && w.icon ? w.icon : null;
      iconUrl.value = icon.value ? makeIconUrl(icon.value) : null;
    } else {
      throw new Error("天气数据格式异常");
    }

    // reverse geocoding to get city name
    try {
      const revUrl = `https://api.openweathermap.org/geo/1.0/reverse?lat=${lat}&lon=${lon}&limit=1&appid=${API_KEY}`;
      const r2 = await fetch(revUrl, { signal });
      if (r2.ok) {
        const arr = await r2.json();
        if (Array.isArray(arr) && arr[0]) {
          const item = arr[0];
          // 优先选择 local_names.zh
          const localNames = item.local_names || {};
          const cityName = localNames.zh || localNames["zh-CN"] || item.name || "";
          const countryCode = item.country || "";
          const countryName = COUNTRY_MAP[countryCode] || countryCode || "";
          const stateRaw = item.state || "";
          const provinceName = PROVINCE_MAP[stateRaw] || stateRaw || "";

          // 组装显示：优先显示 Country · Province · City（当省存在时），否则 Country · City
          if (provinceName) {
            city.value = countryName ? `${countryName} · ${provinceName} · ${cityName}` : `${provinceName} · ${cityName}`;
          } else if (countryName) {
            city.value = `${countryName} · ${cityName}`;
          } else {
            city.value = cityName || "";
          }
        }
      }
    } catch (e) {
      // silently ignore reverse geocoding failure
    }

    loading.value = false;
  } catch (e: any) {
    if (e.name === "AbortError") return;
    error.value = e.message || String(e);
    loading.value = false;
  }
}

function getLocationAndFetch() {
  if (!("geolocation" in navigator)) {
    disabled.value = true;
    loading.value = false;
    return;
  }

  navigator.geolocation.getCurrentPosition(
    (pos) => {
      const lat = pos.coords.latitude;
      const lon = pos.coords.longitude;
      fetchWeather(lat, lon);
    },
    (err) => {
      // permission denied or other errors
      disabled.value = true;
      loading.value = false;
    },
    { enableHighAccuracy: false, timeout: 8000, maximumAge: 1000 * 60 * 10 }
  );
}

onMounted(() => {
  // start with defaults visible while loading
  loading.value = true;
  disabled.value = false;
  error.value = null;

  getLocationAndFetch();
});

onUnmounted(() => {
  if (abortCtrl) abortCtrl.abort();
});

const cityDisplay = computed(() => city.value);
</script>

<style scoped>
.weather-card {
  width: 160px;
  padding: 12px 14px;
  border-radius: 12px;
  background: linear-gradient(
    180deg,
    rgba(255, 255, 255, 0.92),
    rgba(255, 255, 255, 0.86)
  );
  box-shadow: 0 10px 30px rgba(139, 92, 246, 0.08),
    0 2px 8px rgba(0, 0, 0, 0.04);
  color: var(--text-dark);
  border: 1px solid rgba(139, 92, 246, 0.06);
  backdrop-filter: blur(6px);
  display: flex;
  flex-direction: column;
  justify-content: center;
}
.weather-card .top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}
.location {
  font-size: 12px;
  color: rgba(34, 34, 34, 0.7);
}
.cond {
  font-size: 13px;
  font-weight: 600;
  color: var(--accent);
}
.weather-card .bottom {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  gap: 8px;
}
.temp {
  font-size: 26px;
  font-weight: 700;
  color: var(--text-dark);
}
.temp span {
  font-size: 12px;
  margin-left: 2px;
}
.icon {
  width: 44px;
  height: 44px;
}
.msg {
  font-size: 13px;
  color: rgba(34, 34, 34, 0.75);
  text-align: center;
}

/* small screen: hide */
@media (max-width: 720px) {
  .weather-card {
    display: none;
  }
}
</style>
