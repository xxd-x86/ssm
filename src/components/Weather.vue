<template>
  <div class="weather-card">
    <div class="weather-header">
      <h2>天气预报</h2>
      <div class="location-selector">
        <div class="select-group">
          <select v-model="selectedProvince" @change="handleProvinceChange" class="city-select province-select">
            <option value="">选择省份</option>
            <option v-for="province in provinces" :key="province.value" :value="province.value">{{ province.label }}</option>
          </select>
          <select v-model="selectedCity" @change="handleCityChange" class="city-select" :disabled="!selectedProvince">
            <option value="">{{ locationDisplay }}</option>
            <option v-for="city in filteredCities" :key="city.value" :value="city.value">{{ city.label }}</option>
          </select>
        </div>
      </div>
    </div>
    
    <div v-if="loading" class="weather-loading">
      <div class="loader"></div>
      <p>正在获取天气数据...</p>
    </div>
    
    <div v-else-if="error" class="weather-error">
      <svg viewBox="0 0 24 24" width="32" height="32">
        <path fill="#f44336" d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1 15h-2v-2h2v2zm0-4h-2V7h2v6z"></path>
      </svg>
      <p>{{ error }}</p>
      <button @click="fetchWeather" class="retry-btn">重试</button>
    </div>
    
    <div v-else class="weather-content">
      <div class="current-weather">
        <div class="weather-icon" :class="weatherIconClass"></div>
        <div class="weather-details">
          <div class="temp">{{ Math.round(weather.current_condition[0].temp_C) }}°C</div>
          <div class="description">{{ getChineseWeather() }}</div>
          <div class="feels-like">体感温度: {{ Math.round(weather.current_condition[0].FeelsLikeC) }}°C</div>
        </div>
      </div>
      
      <div class="weather-utility">
        <div class="utility-section">
          <div class="utility-header">心情指数</div>
          <div class="utility-content">
            <div class="mood-indicator">
              <div class="mood-emoji">{{ getMoodEmoji() }}</div>
              <div class="mood-text">{{ getMoodText() }}</div>
            </div>
          </div>
        </div>
        
        <div class="utility-section">
          <div class="utility-header">天气趣闻</div>
          <div class="utility-content">
            {{ getWeatherFunFact() }}
          </div>
        </div>
      </div>
      
      <div class="weather-info">
        <div class="info-item">
          <div class="info-label">湿度</div>
          <div class="info-value">{{ weather.current_condition[0].humidity }}%</div>
        </div>
        <div class="info-item">
          <div class="info-label">风速</div>
          <div class="info-value">{{ weather.current_condition[0].windspeedKmph }} km/h</div>
        </div>
        <div class="info-item">
          <div class="info-label">气压</div>
          <div class="info-value">{{ weather.current_condition[0].pressure }} hPa</div>
        </div>
      </div>
      
      <div class="update-time">
        更新时间: {{ formatTime(new Date()) }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Weather',
  data() {
    return {
      weather: null,
      loading: true,
      error: null,
      location: '正在获取位置...',
      selectedProvince: '',
      selectedCity: '',
      queryRetries: 0,
      maxRetries: 1,
      provinces: [
        { label: '北京市', value: 'beijing' },
        { label: '上海市', value: 'shanghai' },
        { label: '天津市', value: 'tianjin' },
        { label: '重庆市', value: 'chongqing' },
        { label: '广东省', value: 'guangdong' },
        { label: '江苏省', value: 'jiangsu' },
        { label: '浙江省', value: 'zhejiang' },
        { label: '四川省', value: 'sichuan' },
        { label: '湖北省', value: 'hubei' },
        { label: '陕西省', value: 'shaanxi' },
        { label: '福建省', value: 'fujian' },
        { label: '山东省', value: 'shandong' },
        { label: '河南省', value: 'henan' },
        { label: '河北省', value: 'hebei' },
        { label: '湖南省', value: 'hunan' },
        { label: '安徽省', value: 'anhui' },
        { label: '辽宁省', value: 'liaoning' },
        { label: '江西省', value: 'jiangxi' },
        { label: '黑龙江省', value: 'heilongjiang' },
        { label: '吉林省', value: 'jilin' },
        { label: '广西壮族自治区', value: 'guangxi' },
        { label: '云南省', value: 'yunnan' },
        { label: '贵州省', value: 'guizhou' },
        { label: '甘肃省', value: 'gansu' },
        { label: '内蒙古自治区', value: 'neimenggu' },
        { label: '山西省', value: 'shanxi' },
        { label: '新疆维吾尔自治区', value: 'xinjiang' },
        { label: '宁夏回族自治区', value: 'ningxia' },
        { label: '海南省', value: 'hainan' },
        { label: '青海省', value: 'qinghai' },
        { label: '西藏自治区', value: 'xizang' },
        { label: '香港特别行政区', value: 'xianggang' },
        { label: '澳门特别行政区', value: 'aomen' },
      ],
      citiesMap: {
        'beijing': [
          { label: '朝阳区', value: 'Beijing,Chaoyang' },
          { label: '海淀区', value: 'Beijing,Haidian' },
          { label: '东城区', value: 'Beijing,Dongcheng' },
          { label: '西城区', value: 'Beijing,Xicheng' },
          { label: '丰台区', value: 'Beijing,Fengtai' },
          { label: '石景山区', value: 'Beijing,Shijingshan' },
          { label: '昌平区', value: 'Beijing,Changping' },
          { label: '大兴区', value: 'Beijing,Daxing' },
          { label: '通州区', value: 'Beijing,Tongzhou' },
          { label: '顺义区', value: 'Beijing,Shunyi' },
          { label: '房山区', value: 'Beijing,Fangshan' },
          { label: '门头沟区', value: 'Beijing,Mentougou' },
          { label: '怀柔区', value: 'Beijing,Huairou' },
          { label: '平谷区', value: 'Beijing,Pinggu' },
          { label: '密云区', value: 'Beijing,Miyun' },
          { label: '延庆区', value: 'Beijing,Yanqing' },
          { label: '北京市', value: 'Beijing' },
        ],
        'shanghai': [
          { label: '浦东新区', value: 'Shanghai,Pudong' },
          { label: '徐汇区', value: 'Shanghai,Xuhui' },
          { label: '长宁区', value: 'Shanghai,Changning' },
          { label: '静安区', value: 'Shanghai,Jingan' },
          { label: '虹口区', value: 'Shanghai,Hongkou' },
          { label: '黄浦区', value: 'Shanghai,Huangpu' },
          { label: '杨浦区', value: 'Shanghai,Yangpu' },
          { label: '普陀区', value: 'Shanghai,Putuo' },
          { label: '闵行区', value: 'Shanghai,Minhang' },
          { label: '宝山区', value: 'Shanghai,Baoshan' },
          { label: '嘉定区', value: 'Shanghai,Jiading' },
          { label: '金山区', value: 'Shanghai,Jinshan' },
          { label: '松江区', value: 'Shanghai,Songjiang' },
          { label: '青浦区', value: 'Shanghai,Qingpu' },
          { label: '奉贤区', value: 'Shanghai,Fengxian' },
          { label: '崇明区', value: 'Shanghai,Chongming' },
          { label: '上海市', value: 'Shanghai' },
        ],
        'guangdong': [
          { label: '广州市', value: 'Guangzhou' },
          { label: '深圳市', value: 'Shenzhen' },
          { label: '珠海市', value: 'Zhuhai' },
          { label: '东莞市', value: 'Dongguan' },
          { label: '佛山市', value: 'Foshan' },
          { label: '惠州市', value: 'Huizhou' },
          { label: '中山市', value: 'Zhongshan' },
          { label: '汕头市', value: 'Shantou' },
          { label: '湛江市', value: 'Zhanjiang' },
          { label: '江门市', value: 'Jiangmen' },
          { label: '茂名市', value: 'Maoming' },
          { label: '肇庆市', value: 'Zhaoqing' },
          { label: '梅州市', value: 'Meizhou' },
          { label: '汕尾市', value: 'Shanwei' },
          { label: '韶关市', value: 'Shaoguan' },
          { label: '阳江市', value: 'Yangjiang' },
          { label: '河源市', value: 'Heyuan' },
          { label: '清远市', value: 'Qingyuan' },
          { label: '潮州市', value: 'Chaozhou' },
          { label: '揭阳市', value: 'Jieyang' },
          { label: '云浮市', value: 'Yunfu' },
        ],
        'jiangsu': [
          { label: '南京市', value: 'Nanjing' },
          { label: '苏州市', value: 'Suzhou' },
          { label: '无锡市', value: 'Wuxi' },
          { label: '常州市', value: 'Changzhou' },
          { label: '扬州市', value: 'Yangzhou' },
          { label: '南通市', value: 'Nantong' },
        ],
        'zhejiang': [
          { label: '杭州市', value: 'Hangzhou' },
          { label: '宁波市', value: 'Ningbo' },
          { label: '温州市', value: 'Wenzhou' },
          { label: '嘉兴市', value: 'Jiaxing' },
          { label: '绍兴市', value: 'Shaoxing' },
        ],
        'sichuan': [
          { label: '成都市', value: 'Chengdu' },
          { label: '绵阳市', value: 'Mianyang' },
          { label: '自贡市', value: 'Zigong' },
          { label: '内江市', value: 'Neijiang' },
        ],
        'hubei': [
          { label: '武汉市', value: 'Wuhan' },
          { label: '宜昌市', value: 'Yichang' },
          { label: '黄石市', value: 'Huangshi' },
          { label: '襄阳市', value: 'Xiangyang' },
        ],
        'shaanxi': [
          { label: '西安市', value: 'Xi\'an' },
          { label: '宝鸡市', value: 'Baoji' },
          { label: '咸阳市', value: 'Xianyang' },
        ],
        'chongqing': [
          { label: '渝中区', value: 'Chongqing,Yuzhong' },
          { label: '江北区', value: 'Chongqing,Jiangbei' },
          { label: '九龙坡区', value: 'Chongqing,Jiulongpo' },
          { label: '南岸区', value: 'Chongqing,Nanan' },
          { label: '沙坪坝区', value: 'Chongqing,Shapingba' },
          { label: '渝北区', value: 'Chongqing,Yubei' },
          { label: '巴南区', value: 'Chongqing,Banan' },
          { label: '大渡口区', value: 'Chongqing,Dadukou' },
          { label: '北碚区', value: 'Chongqing,Beibei' },
          { label: '万州区', value: 'Chongqing,Wanzhou' },
          { label: '涪陵区', value: 'Chongqing,Fuling' },
          { label: '黔江区', value: 'Chongqing,Qianjiang' },
          { label: '永川区', value: 'Chongqing,Yongchuan' },
          { label: '合川区', value: 'Chongqing,Hechuan' },
          { label: '江津区', value: 'Chongqing,Jiangjin' },
          { label: '南川区', value: 'Chongqing,Nanchuan' },
          { label: '綦江区', value: 'Chongqing,Qijiang' },
          { label: '大足区', value: 'Chongqing,Dazu' },
          { label: '璧山区', value: 'Chongqing,Bishan' },
          { label: '铜梁区', value: 'Chongqing,Tongliang' },
          { label: '潼南区', value: 'Chongqing,Tongnan' },
          { label: '荣昌区', value: 'Chongqing,Rongchang' },
          { label: '开州区', value: 'Chongqing,Kaizhou' },
          { label: '梁平区', value: 'Chongqing,Liangping' },
          { label: '武隆区', value: 'Chongqing,Wulong' },
          { label: '城口县', value: 'Chongqing,Chengkou' },
          { label: '丰都县', value: 'Chongqing,Fengdu' },
          { label: '垫江县', value: 'Chongqing,Dianjiang' },
          { label: '忠县', value: 'Chongqing,Zhongxian' },
          { label: '云阳县', value: 'Chongqing,Yunyang' },
          { label: '奉节县', value: 'Chongqing,Fengjie' },
          { label: '巫山县', value: 'Chongqing,Wushan' },
          { label: '巫溪县', value: 'Chongqing,Wuxi' },
          { label: '石柱土家族自治县', value: 'Chongqing,Shizhu' },
          { label: '秀山土家族苗族自治县', value: 'Chongqing,Xiushan' },
          { label: '酉阳土家族苗族自治县', value: 'Chongqing,Youyang' },
          { label: '彭水苗族土家族自治县', value: 'Chongqing,Pengshui' },
          { label: '重庆市', value: 'Chongqing' },
        ],
        'tianjin': [
          { label: '和平区', value: 'Tianjin,Heping' },
          { label: '河东区', value: 'Tianjin,Hedong' },
          { label: '河西区', value: 'Tianjin,Hexi' },
          { label: '天津市', value: 'Tianjin' },
        ],
        'fujian': [
          { label: '福州市', value: 'Fuzhou' },
          { label: '厦门市', value: 'Xiamen' },
          { label: '泉州市', value: 'Quanzhou' },
        ],
        'shandong': [
          { label: '济南市', value: 'Jinan' },
          { label: '青岛市', value: 'Qingdao' },
          { label: '烟台市', value: 'Yantai' },
          { label: '威海市', value: 'Weihai' },
        ],
        'henan': [
          { label: '郑州市', value: 'Zhengzhou' },
          { label: '洛阳市', value: 'Luoyang' },
          { label: '开封市', value: 'Kaifeng' },
        ],
        'hebei': [
          { label: '石家庄市', value: 'Shijiazhuang' },
          { label: '唐山市', value: 'Tangshan' },
          { label: '保定市', value: 'Baoding' },
        ],
        'hunan': [
          { label: '长沙市', value: 'Changsha' },
          { label: '株洲市', value: 'Zhuzhou' },
          { label: '湘潭市', value: 'Xiangtan' },
        ],
        'anhui': [
          { label: '合肥市', value: 'Hefei' },
          { label: '芜湖市', value: 'Wuhu' },
          { label: '安庆市', value: 'Anqing' },
        ],
        'liaoning': [
          { label: '沈阳市', value: 'Shenyang' },
          { label: '大连市', value: 'Dalian' },
          { label: '鞍山市', value: 'Anshan' },
        ],
        'jiangxi': [
          { label: '南昌市', value: 'Nanchang' },
          { label: '九江市', value: 'Jiujiang' },
          { label: '赣州市', value: 'Ganzhou' },
          { label: '宜春市', value: 'Yichun,Jiangxi' },
          { label: '吉安市', value: 'Ji\'an' },
          { label: '上饶市', value: 'Shangrao' },
          { label: '抚州市', value: 'Fuzhou,Jiangxi' },
          { label: '景德镇市', value: 'Jingdezhen' },
          { label: '萍乡市', value: 'Pingxiang' },
          { label: '新余市', value: 'Xinyu' },
          { label: '鹰潭市', value: 'Yingtan' },
        ],
        'heilongjiang': [
          { label: '哈尔滨市', value: 'Harbin' },
          { label: '大庆市', value: 'Daqing' },
          { label: '齐齐哈尔市', value: 'Qiqihar' },
          { label: '佳木斯市', value: 'Jiamusi' },
          { label: '牡丹江市', value: 'Mudanjiang' },
          { label: '绥化市', value: 'Suihua' },
          { label: '黑河市', value: 'Heihe' },
          { label: '鹤岗市', value: 'Hegang' },
          { label: '双鸭山市', value: 'Shuangyashan' },
          { label: '伊春市', value: 'Yichun,Heilongjiang' },
          { label: '七台河市', value: 'Qitaihe' },
          { label: '鸡西市', value: 'Jixi' },
          { label: '大兴安岭地区', value: 'Daxinganling' },
        ],
        'jilin': [
          { label: '长春市', value: 'Changchun' },
          { label: '吉林市', value: 'Jilin' },
          { label: '延边朝鲜族自治州', value: 'Yanbian' },
          { label: '四平市', value: 'Siping' },
          { label: '通化市', value: 'Tonghua' },
          { label: '白山市', value: 'Baishan' },
          { label: '辽源市', value: 'Liaoyuan' },
          { label: '松原市', value: 'Songyuan' },
          { label: '白城市', value: 'Baicheng' },
        ],
        'guangxi': [
          { label: '南宁市', value: 'Nanning' },
          { label: '柳州市', value: 'Liuzhou' },
          { label: '桂林市', value: 'Guilin' },
          { label: '北海市', value: 'Beihai' },
          { label: '玉林市', value: 'Yulin,Guangxi' },
          { label: '钦州市', value: 'Qinzhou' },
          { label: '贵港市', value: 'Guigang' },
          { label: '防城港市', value: 'Fangchenggang' },
          { label: '百色市', value: 'Baise' },
          { label: '梧州市', value: 'Wuzhou' },
          { label: '贺州市', value: 'Hezhou' },
          { label: '河池市', value: 'Hechi' },
          { label: '来宾市', value: 'Laibin' },
          { label: '崇左市', value: 'Chongzuo' },
        ],
        'yunnan': [
          { label: '昆明市', value: 'Kunming' },
          { label: '大理白族自治州', value: 'Dali' },
          { label: '丽江市', value: 'Lijiang' },
          { label: '西双版纳傣族自治州', value: 'Xishuangbanna' },
          { label: '曲靖市', value: 'Qujing' },
          { label: '玉溪市', value: 'Yuxi' },
          { label: '保山市', value: 'Baoshan' },
          { label: '昭通市', value: 'Zhaotong' },
          { label: '普洱市', value: 'Puer' },
          { label: '临沧市', value: 'Lincang' },
          { label: '楚雄彝族自治州', value: 'Chuxiong' },
          { label: '红河哈尼族彝族自治州', value: 'Honghe' },
          { label: '文山壮族苗族自治州', value: 'Wenshan' },
          { label: '德宏傣族景颇族自治州', value: 'Dehong' },
          { label: '怒江傈僳族自治州', value: 'Nujiang' },
          { label: '迪庆藏族自治州', value: 'Diqing' },
        ],
        'guizhou': [
          { label: '贵阳市', value: 'Guiyang' },
          { label: '遵义市', value: 'Zunyi' },
          { label: '安顺市', value: 'Anshun' },
          { label: '六盘水市', value: 'Liupanshui' },
          { label: '毕节市', value: 'Bijie' },
          { label: '铜仁市', value: 'Tongren' },
          { label: '黔东南苗族侗族自治州', value: 'Qiandongnan' },
          { label: '黔南布依族苗族自治州', value: 'Qiannan' },
          { label: '黔西南布依族苗族自治州', value: 'Qianxinan' },
        ],
        'gansu': [
          { label: '兰州市', value: 'Lanzhou' },
          { label: '天水市', value: 'Tianshui' },
          { label: '酒泉市', value: 'Jiuquan' },
          { label: '张掖市', value: 'Zhangye' },
          { label: '嘉峪关市', value: 'Jiayuguan' },
          { label: '金昌市', value: 'Jinchang' },
          { label: '白银市', value: 'Baiyin' },
          { label: '武威市', value: 'Wuwei' },
          { label: '平凉市', value: 'Pingliang' },
          { label: '庆阳市', value: 'Qingyang' },
          { label: '定西市', value: 'Dingxi' },
          { label: '陇南市', value: 'Longnan' },
          { label: '临夏回族自治州', value: 'Linxia' },
          { label: '甘南藏族自治州', value: 'Gannan' },
        ],
        'neimenggu': [
          { label: '呼和浩特市', value: 'Hohhot' },
          { label: '包头市', value: 'Baotou' },
          { label: '鄂尔多斯市', value: 'Ordos' },
          { label: '赤峰市', value: 'Chifeng' },
          { label: '通辽市', value: 'Tongliao' },
          { label: '呼伦贝尔市', value: 'Hulunbuir' },
          { label: '巴彦淖尔市', value: 'Bayannur' },
          { label: '乌兰察布市', value: 'Ulanqab' },
          { label: '锡林郭勒盟', value: 'Xilingol' },
          { label: '兴安盟', value: 'Hinggan' },
          { label: '乌海市', value: 'Wuhai' },
          { label: '阿拉善盟', value: 'Alxa' },
        ],
        'shanxi': [
          { label: '太原市', value: 'Taiyuan' },
          { label: '大同市', value: 'Datong' },
          { label: '运城市', value: 'Yuncheng' },
          { label: '临汾市', value: 'Linfen' },
          { label: '晋中市', value: 'Jinzhong' },
          { label: '长治市', value: 'Changzhi' },
          { label: '忻州市', value: 'Xinzhou' },
          { label: '吕梁市', value: 'Lvliang' },
          { label: '晋城市', value: 'Jincheng' },
          { label: '朔州市', value: 'Shuozhou' },
          { label: '阳泉市', value: 'Yangquan' },
        ],
        'xinjiang': [
          { label: '乌鲁木齐市', value: 'Urumqi' },
          { label: '喀什地区', value: 'Kashgar' },
          { label: '伊犁哈萨克自治州', value: 'Ili' },
          { label: '昌吉回族自治州', value: 'Changji' },
          { label: '阿克苏地区', value: 'Aksu' },
          { label: '哈密市', value: 'Hami' },
          { label: '吐鲁番市', value: 'Turpan' },
          { label: '巴音郭楞蒙古自治州', value: 'Bayingol' },
          { label: '克拉玛依市', value: 'Karamay' },
          { label: '博尔塔拉蒙古自治州', value: 'Bortala' },
          { label: '塔城地区', value: 'Tacheng' },
          { label: '阿勒泰地区', value: 'Altay' },
          { label: '克孜勒苏柯尔克孜自治州', value: 'Kizilsu' },
          { label: '和田地区', value: 'Hotan' },
        ],
        'ningxia': [
          { label: '银川市', value: 'Yinchuan' },
          { label: '石嘴山市', value: 'Shizuishan' },
          { label: '吴忠市', value: 'Wuzhong' },
          { label: '固原市', value: 'Guyuan' },
          { label: '中卫市', value: 'Zhongwei' },
        ],
        'hainan': [
          { label: '海口市', value: 'Haikou' },
          { label: '三亚市', value: 'Sanya' },
          { label: '三沙市', value: 'Sansha' },
          { label: '儋州市', value: 'Danzhou' },
          { label: '五指山市', value: 'Wuzhishan' },
          { label: '文昌市', value: 'Wenchang' },
          { label: '琼海市', value: 'Qionghai' },
          { label: '万宁市', value: 'Wanning' },
          { label: '东方市', value: 'Dongfang' },
        ],
        'qinghai': [
          { label: '西宁市', value: 'Xining' },
          { label: '海东市', value: 'Haidong' },
          { label: '海西蒙古族藏族自治州', value: 'Haixi' },
          { label: '玉树藏族自治州', value: 'Yushu' },
          { label: '海北藏族自治州', value: 'Haibei' },
          { label: '果洛藏族自治州', value: 'Golog' },
          { label: '海南藏族自治州', value: 'Hainan,Qinghai' },
          { label: '黄南藏族自治州', value: 'Huangnan' },
        ],
        'xizang': [
          { label: '拉萨市', value: 'Lhasa' },
          { label: '日喀则市', value: 'Shigatse' },
          { label: '昌都市', value: 'Qamdo' },
          { label: '林芝市', value: 'Nyingchi' },
          { label: '山南市', value: 'Shannan' },
          { label: '那曲市', value: 'Nagqu' },
          { label: '阿里地区', value: 'Ngari' },
        ],
        'xianggang': [
          { label: '中西区', value: 'Central and Western,Hong Kong' },
          { label: '湾仔区', value: 'Wan Chai,Hong Kong' },
          { label: '东区', value: 'Eastern,Hong Kong' },
          { label: '南区', value: 'Southern,Hong Kong' },
          { label: '油尖旺区', value: 'Yau Tsim Mong,Hong Kong' },
          { label: '深水埗区', value: 'Sham Shui Po,Hong Kong' },
          { label: '九龙城区', value: 'Kowloon City,Hong Kong' },
          { label: '黄大仙区', value: 'Wong Tai Sin,Hong Kong' },
          { label: '观塘区', value: 'Kwun Tong,Hong Kong' },
          { label: '荃湾区', value: 'Tsuen Wan,Hong Kong' },
          { label: '屯门区', value: 'Tuen Mun,Hong Kong' },
          { label: '元朗区', value: 'Yuen Long,Hong Kong' },
          { label: '北区', value: 'North,Hong Kong' },
          { label: '大埔区', value: 'Tai Po,Hong Kong' },
          { label: '沙田区', value: 'Sha Tin,Hong Kong' },
          { label: '西贡区', value: 'Sai Kung,Hong Kong' },
          { label: '离岛区', value: 'Islands,Hong Kong' },
          { label: '香港', value: 'Hong Kong' },
        ],
        'aomen': [
          { label: '花地玛堂区', value: 'Nossa Senhora de Fatima,Macau' },
          { label: '圣安多尼堂区', value: 'Santo Antonio,Macau' },
          { label: '大堂区', value: 'Sé,Macau' },
          { label: '望德堂区', value: 'Sao Lazaro,Macau' },
          { label: '风顺堂区', value: 'Sao Lourenco,Macau' },
          { label: '嘉模堂区', value: 'Nossa Senhora do Carmo,Macau' },
          { label: '圣方济各堂区', value: 'Sao Francisco Xavier,Macau' },
          { label: '路氹城', value: 'Cotai,Macau' },
          { label: '澳门', value: 'Macau' },
        ]
      },
      weatherIconMap: {
        '01d': 'clear-day.svg',
        '01n': 'clear-night.svg',
        '02d': 'partly-cloudy-day.svg',
        '02n': 'partly-cloudy-night.svg',
        '03d': 'cloudy.svg',
        '03n': 'cloudy.svg',
        '04d': 'overcast-day.svg',
        '04n': 'overcast-night.svg',
        '09d': 'rain.svg',
        '09n': 'rain.svg',
        '10d': 'partly-cloudy-day-rain.svg',
        '10n': 'partly-cloudy-night-rain.svg',
        '11d': 'thunderstorms-day.svg',
        '11n': 'thunderstorms-night.svg',
        '13d': 'snow.svg',
        '13n': 'snow.svg',
        '50d': 'mist.svg',
        '50n': 'mist.svg',
        'sunny': 'clear-day.svg',
        'clear': 'clear-night.svg',
        'partly-cloudy': 'partly-cloudy-day.svg',
        'cloudy': 'cloudy.svg', 
        'overcast': 'overcast-day.svg',
        'mist': 'mist.svg',
        'fog': 'fog.svg',
        'light-rain': 'drizzle.svg',
        'moderate-rain': 'rain.svg',
        'heavy-rain': 'rain.svg',
        'thunderstorm': 'thunderstorms.svg',
        'snow': 'snow.svg',
        'sleet': 'sleet.svg',
        'default': 'clear-day.svg'
      }
    };
  },
  computed: {
    locationDisplay() {
      return this.location || '选择城市';
    },
    filteredCities() {
      return this.selectedProvince ? this.citiesMap[this.selectedProvince] || [] : [];
    },
    weatherIconClass() {
      if (!this.weather) return 'wi-day-sunny';
      
      const condition = this.weather.current_condition?.[0]?.weatherDesc?.[0]?.value || '';
      
      // 根据天气描述映射到图标
      if (condition.includes('晴') || condition.includes('Clear')) {
        return 'wi-day-sunny';
      } else if (condition.includes('多云') || condition.includes('Partly cloudy')) {
        return 'wi-day-cloudy';
      } else if (condition.includes('阴') || condition.includes('Cloudy') || condition.includes('Overcast')) {
        return 'wi-cloudy';
      } else if (condition.includes('雨') || condition.includes('Rain') || condition.includes('Drizzle')) {
        if (condition.includes('雷') || condition.includes('Thunder')) {
          return 'wi-thunderstorm';
        }
        return 'wi-rain';
      } else if (condition.includes('雪') || condition.includes('Snow')) {
        return 'wi-snow';
      } else if (condition.includes('雾') || condition.includes('Fog') || condition.includes('Mist')) {
        return 'wi-fog';
      }
      
      // 默认
      return 'wi-day-sunny';
    }
  },
  mounted() {
    this.getUserLocation();
  },
  methods: {
    getUserLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            this.fetchWeatherByCoords(position.coords.latitude, position.coords.longitude);
          },
          (error) => {
            console.error('定位获取失败:', error);
            this.fetchWeatherByCity('Beijing');
          }
        );
      } else {
        this.fetchWeatherByCity('Beijing');
      }
    },
    async fetchWeatherByCoords(lat, lon) {
      try {
        this.loading = true;
        this.error = null;
        this.queryRetries = 0;
        
        const response = await fetch(`https://wttr.in/${lat},${lon}?format=j1`);
        
        if (!response.ok) {
          throw new Error('天气数据获取失败');
        }
        
        const data = await response.json();
        
        // 检查返回的数据是否有效
        if (!this.isValidWeatherData(data)) {
          throw new Error('返回的天气数据无效');
        }
        
        this.weather = data;
        this.location = data.nearest_area?.[0]?.areaName?.[0]?.value || '当前位置';
        this.loading = false;
      } catch (error) {
        console.error('获取天气信息失败:', error);
        this.error = '获取天气信息失败，请稍后重试';
        this.loading = false;
      }
    },
    async fetchWeatherByCity(city) {
      try {
        this.loading = true;
        this.error = null;
        this.queryRetries = 0;
        
        const response = await fetch(`https://wttr.in/${city}?format=j1`);
        
        if (!response.ok) {
          throw new Error('天气数据获取失败');
        }
        
        const data = await response.json();
        
        // 检查返回的数据是否有效
        if (!this.isValidWeatherData(data)) {
          // 如果是地区查询失败，尝试回退到上一级城市
          if (this.queryRetries < this.maxRetries) {
            this.queryRetries++;
            
            // 提取主城市名称（如果格式是City,District）
            const mainCity = city.split(',')[0];
            if (mainCity !== city) {
              console.log('精确地点查询失败，尝试使用主城市:', mainCity);
              return this.fetchWeatherByCity(mainCity);
            }
          }
          
          throw new Error('找不到该地点的天气数据');
        }
        
        this.weather = data;
        
        // 提取并设置位置名称
        let locationName;
        if (city.includes(',')) {
          // 如果是区县级查询，显示完整地点名
          const parts = city.split(',');
          const cityName = this.getCityDisplayName(parts[0]);
          const districtName = this.getDistrictDisplayName(parts[1], parts[0]);
          locationName = `${cityName} ${districtName}`;
        } else {
          locationName = data.nearest_area?.[0]?.areaName?.[0]?.value || city;
        }
        
        this.location = locationName;
        this.loading = false;
      } catch (error) {
        console.error('获取天气信息失败:', error);
        this.error = '找不到该地点的天气数据';
        this.loading = false;
      }
    },
    getCityDisplayName(cityCode) {
      // 查找城市的显示名称
      for (const province in this.citiesMap) {
        const city = this.citiesMap[province].find(c => c.value.split(',')[0] === cityCode);
        if (city) return city.label;
      }
      return cityCode;
    },
    getDistrictDisplayName(districtCode, cityCode) {
      // 这里可以添加区县名称的映射
      return districtCode;
    },
    isValidWeatherData(data) {
      return data && 
             data.current_condition && 
             data.current_condition.length > 0 && 
             data.current_condition[0].temp_C !== undefined;
    },
    formatTime(date) {
      const options = { hour: '2-digit', minute: '2-digit' };
      return date.toLocaleTimeString('zh-CN', options);
    },
    isDaytime() {
      const hour = new Date().getHours();
      return hour >= 6 && hour < 18;
    },
    getChineseWeather() {
      if (!this.weather || !this.weather.current_condition || this.weather.current_condition.length === 0) {
        return '晴天';
      }
      
      return this.weather.current_condition[0].weatherDesc[0].value || '晴天';
    },
    getAstronomyData(type) {
      if (!this.weather || !this.weather.weather || !this.weather.weather[0] || !this.weather.weather[0].astronomy || !this.weather.weather[0].astronomy[0]) {
        return type === 'sunrise' ? '06:00' : '18:00';
      }
      
      return this.weather.weather[0].astronomy[0][type] || (type === 'sunrise' ? '06:00' : '18:00');
    },
    getTravelSuggestion() {
      if (!this.weather || !this.weather.current_condition || this.weather.current_condition.length === 0) {
        return '今日天气良好，适宜出行';
      }
      
      const condition = this.weather.current_condition[0].weatherDesc[0].value.toLowerCase();
      const temp = parseFloat(this.weather.current_condition[0].temp_C);
      
      if (condition.includes('rain') || condition.includes('雨')) {
        return '带好雨具，注意防雨';
      } else if (condition.includes('snow') || condition.includes('雪')) {
        return '注意防寒保暖，路面可能湿滑';
      } else if (condition.includes('fog') || condition.includes('mist') || condition.includes('雾')) {
        return '能见度较低，驾车注意安全';
      } else if (condition.includes('thunder') || condition.includes('雷')) {
        return '雷雨天气，尽量避免户外活动';
      } else if (temp > 30) {
        return '气温较高，注意防暑降温';
      } else if (temp < 5) {
        return '气温较低，注意保暖';
      } else if (condition.includes('sunny') || condition.includes('clear') || condition.includes('晴')) {
        return '天气晴好，适宜户外活动';
      } else {
        return '今日天气良好，适宜出行';
      }
    },
    fetchWeather() {
      this.getUserLocation();
    },
    handleProvinceChange() {
      this.selectedCity = '';
    },
    handleCityChange() {
      if (this.selectedCity) {
        this.fetchWeatherByCity(this.selectedCity);
      }
    },
    getMoodEmoji() {
      if (!this.weather || !this.weather.current_condition || this.weather.current_condition.length === 0) {
        return '😊';
      }
      
      const condition = this.weather.current_condition[0].weatherDesc[0].value.toLowerCase();
      const temp = parseFloat(this.weather.current_condition[0].temp_C);
      
      if (condition.includes('thunder') || condition.includes('雷')) {
        return '😱';
      } else if (condition.includes('rain') || condition.includes('雨')) {
        return '😔';
      } else if (condition.includes('snow') || condition.includes('雪')) {
        return '😍';
      } else if (condition.includes('fog') || condition.includes('mist') || condition.includes('雾')) {
        return '😶';
      } else if (condition.includes('sunny') || condition.includes('clear') || condition.includes('晴')) {
        return temp > 30 ? '🥵' : '😎';
      } else if (condition.includes('cloudy') || condition.includes('overcast') || condition.includes('阴')) {
        return '😌';
      } else if (temp > 30) {
        return '🥵';
      } else if (temp < 5) {
        return '🥶';
      }
      
      return '😊';
    },
    getMoodText() {
      if (!this.weather || !this.weather.current_condition || this.weather.current_condition.length === 0) {
        return '心情愉悦';
      }
      
      const condition = this.weather.current_condition[0].weatherDesc[0].value.toLowerCase();
      const temp = parseFloat(this.weather.current_condition[0].temp_C);
      
      if (condition.includes('thunder') || condition.includes('雷')) {
        return '紧张不安';
      } else if (condition.includes('rain') || condition.includes('雨')) {
        if (condition.includes('light') || condition.includes('小')) {
          return '略感忧郁';
        }
        return '心情低落';
      } else if (condition.includes('snow') || condition.includes('雪')) {
        return '兴奋激动';
      } else if (condition.includes('fog') || condition.includes('mist') || condition.includes('雾')) {
        return '思绪迷茫';
      } else if (condition.includes('sunny') || condition.includes('clear') || condition.includes('晴')) {
        if (temp > 30) {
          return '热情似火';
        }
        return '神清气爽';
      } else if (condition.includes('cloudy') || condition.includes('overcast') || condition.includes('阴')) {
        return '平静舒适';
      } else if (temp > 30) {
        return '烦躁不安';
      } else if (temp < 5) {
        return '瑟瑟发抖';
      }
      
      const moods = ['心旷神怡', '轻松愉快', '活力四射', '心情舒畅', '惬意安然'];
      return moods[Math.floor(Math.random() * moods.length)];
    },
    getWeatherFunFact() {
      if (!this.weather || !this.weather.current_condition || this.weather.current_condition.length === 0) {
        return this.getRandomFunFact();
      }
      
      const condition = this.weather.current_condition[0].weatherDesc[0].value.toLowerCase();
      
      const funFacts = {
        rain: [
          '你知道吗？最大的雨滴直径可达8毫米，而普通雨滴仅为2毫米。',
          '世界上最多雨的地方是印度的莫辛拉姆，年均降雨量11,871毫米！',
          '雨中漫步能够提高创造力，这是科学家们证实的哦！',
          '雨滴落下时并不是泪珠形状，而更像一个小降落伞。'
        ],
        snow: [
          '世界上没有两片完全相同的雪花，这是因为每片雪花形成时的温度和湿度都不同。',
          '雪实际上是无色透明的，看起来是白色是因为阳光反射了所有的颜色。',
          '日本长野县曾记录到世界最厚的积雪，达到惊人的11.82米！',
          '雪能够隔音！新落下的雪能吸收声音，让四周变得格外安静。'
        ],
        sunny: [
          '阳光照射到地球需要大约8分钟20秒的时间。',
          '在地球的大气层外，太阳能量是地面接收到的30%。',
          '晴朗天空是蓝色的，是因为空气分子散射了更多的蓝光。',
          '你知道吗？阳光除了让人心情愉悦，还能促进体内维生素D的合成。'
        ],
        cloudy: [
          '一朵普通的积云重量可超过500吨，相当于100头大象的重量！',
          '世界上最常见的云是积云，它看起来像是棉花糖一样蓬松。',
          '云的高度可以告诉我们天气变化，高云通常预示着好天气。',
          '在特定条件下，云可以形成奇特的形状，如飞碟云、波浪云等。'
        ],
        thunder: [
          '闪电的温度可达30,000°C，比太阳表面还要热5倍！',
          '一道闪电的能量足以照亮全球100万个灯泡。',
          '闪电实际是从地面向上而不是从云向下的，你看到的只是回程！',
          '全球每秒钟约有44-55次闪电，每天约有800万道闪电。'
        ],
        fog: [
          '雾实际上就是接触地面的云层，它们由相同的水滴组成。',
          '世界上雾最多的地方是加拿大纽芬兰大银行，每年有超过200天的大雾天气。',
          '特浓的雾可以使能见度降低到不足1米，这被称为"豌豆汤式"大雾。',
          '雾可以作为水源！一些干旱地区使用特殊网收集雾中的水分。'
        ],
        default: [
          '地球上的水资源约有97.5%是咸水，只有2.5%是淡水。',
          '蓝鲸的"喷水"其实是呼出的热气遇冷空气凝结形成的水汽，类似我们在寒冷天气呼气时看到的现象。',
          '亚马逊雨林产生地球20%的氧气，被称为"地球之肺"。',
          '日本有一种用来预测天气的小玩具"晴天娃娃"，会随着湿度变化而改变颜色。',
          '虹膜通常是七彩的，但在极其罕见的情况下，还能看到月虹和雾虹。'
        ]
      };
      
      let category = 'default';
      
      if (condition.includes('rain') || condition.includes('雨')) {
        category = 'rain';
      } else if (condition.includes('snow') || condition.includes('雪')) {
        category = 'snow';
      } else if (condition.includes('sunny') || condition.includes('clear') || condition.includes('晴')) {
        category = 'sunny';
      } else if (condition.includes('cloudy') || condition.includes('overcast') || condition.includes('阴')) {
        category = 'cloudy';
      } else if (condition.includes('thunder') || condition.includes('雷')) {
        category = 'thunder';
      } else if (condition.includes('fog') || condition.includes('mist') || condition.includes('雾')) {
        category = 'fog';
      }
      
      const facts = funFacts[category] || funFacts.default;
      return facts[Math.floor(Math.random() * facts.length)];
    },
    getRandomFunFact() {
      const allFacts = [
        '虹通常是七彩的，但在极其罕见的情况下，还能看到月虹和雾虹。',
        '史上最大的冰雹重达1公斤，发生在1986年的孟加拉国。',
        '日本有一种用来预测天气的小玩具"晴天娃娃"，会随着湿度变化而改变颜色。',
        '雷暴期间，男性被闪电击中的几率是女性的四倍。',
        '珠穆朗玛峰顶端的气温从未超过零度，即使在夏天。'
      ];
      
      return allFacts[Math.floor(Math.random() * allFacts.length)];
    }
  }
};
</script>

<style scoped>
.weather-card {
  background: linear-gradient(to bottom right, #2193b0, #6dd5ed);
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
  color: white;
  padding: 20px;
  overflow: hidden;
  position: relative;
}

.weather-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(120deg, rgba(255,255,255,0.2) 0%, rgba(255,255,255,0) 70%);
  z-index: 1;
  pointer-events: none;
}

.weather-header {
  margin-bottom: 15px;
}

.weather-header h2 {
  margin: 0 0 8px;
  font-size: 22px;
  font-weight: 600;
}

.location-selector {
  margin-top: 10px;
  width: 100%;
}

.select-group {
  display: flex;
  gap: 10px;
  width: 100%;
}

.province-select {
  flex: 2;
}

.city-select {
  flex: 3;
  width: 100%;
  padding: 8px 12px;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  background-color: rgba(255, 255, 255, 0.15);
  color: white;
  font-size: 16px;
  appearance: none;
  cursor: pointer;
  outline: none;
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="white" width="18px" height="18px"><path d="M7 10l5 5 5-5z"/></svg>');
  background-repeat: no-repeat;
  background-position: right 12px center;
  padding-right: 40px;
}

.city-select:hover:not(:disabled) {
  background-color: rgba(255, 255, 255, 0.25);
}

.city-select:focus {
  border-color: rgba(255, 255, 255, 0.5);
}

.city-select:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.city-select option {
  background-color: #2193b0;
  color: white;
}

.weather-loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 180px;
}

.loader {
  border: 3px solid rgba(255,255,255,0.3);
  border-top: 3px solid white;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  animation: spin 1s linear infinite;
  margin-bottom: 15px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.weather-error {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 180px;
  text-align: center;
}

.retry-btn {
  background: rgba(255, 255, 255, 0.2);
  border: none;
  border-radius: 20px;
  padding: 7px 20px;
  color: white;
  font-size: 14px;
  cursor: pointer;
  margin-top: 15px;
  transition: background 0.3s;
}

.retry-btn:hover {
  background: rgba(255, 255, 255, 0.3);
}

.current-weather {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.weather-icon {
  width: 80px;
  height: 80px;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
}

.wi-thunderstorm {
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64"><g fill="white"><path d="M41.7 36.3c-.2 0-.4-.1-.6-.2-.5-.3-.7-.9-.4-1.4l3.5-6.4h-4.5c-.4 0-.7-.2-.9-.5-.2-.3-.2-.7 0-1l5-10c.2-.4.6-.6 1-.6h3.8c.4 0 .8.2.9.6.2.4.1.8-.1 1.1L45.9 24h4.6c.4 0 .8.2.9.6.2.4.1.8-.2 1.1l-8.9 10c-.1.4-.4.6-.6.6z"/><path d="M46.7 38.9c-.2 0-.4-.1-.6-.2-.5-.3-.7-.9-.4-1.4l2.1-3.9h-2.7c-.4 0-.7-.2-.9-.5-.2-.3-.2-.7 0-1l3-6c.2-.4.6-.6 1-.6h2.3c.4 0 .8.2.9.6.2.4.1.8-.1 1.1l-2 3.5h2.7c.4 0 .8.2.9.6.2.4.1.8-.2 1.1l-5.3 6c-.3.5-.5.7-.7.7z"/><path d="M28.7 46.9c-.2 0-.4-.1-.6-.2-.5-.3-.7-.9-.4-1.4l3.5-6.4h-4.5c-.4 0-.7-.2-.9-.5-.2-.3-.2-.7 0-1l5-10c.2-.4.6-.6 1-.6h3.8c.4 0 .8.2.9.6.2.4.1.8-.1 1.1L32.9 34.6h4.6c.4 0 .8.2.9.6.2.4.1.8-.2 1.1l-8.9 10c-.1.4-.3.6-.6.6z"/></g></svg>');
}

.wi-drizzle {
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64"><g fill="white"><path d="M26.2 46.2c-.3 0-.6-.2-.8-.5-.2-.4-.1-.9.3-1.2l1.8-1.1c.4-.2.9-.1 1.2.3.2.4.1.9-.3 1.2l-1.8 1.1c-.2.2-.3.2-.4.2zM32.2 42.2c-.3 0-.6-.2-.8-.5-.2-.4-.1-.9.3-1.2l1.8-1.1c.4-.2.9-.1 1.2.3.2.4.1.9-.3 1.2l-1.8 1.1c-.1.2-.3.2-.4.2zM38.2 38.2c-.3 0-.6-.2-.8-.5-.2-.4-.1-.9.3-1.2l1.8-1.1c.4-.2.9-.1 1.2.3.2.4.1.9-.3 1.2l-1.8 1.1c-.1.2-.2.2-.4.2z"/></g></svg>');
}

.wi-rain {
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64"><g fill="white"><path d="M25.6 46.6c-.3 0-.6-.2-.8-.5l-2.2-4.5c-.2-.4 0-.9.4-1.1.4-.2.9 0 1.1.4l2.2 4.5c.2.4 0 .9-.4 1.1 0 .1-.2.1-.3.1zM32.6 46.6c-.3 0-.6-.2-.8-.5l-2.2-4.5c-.2-.4 0-.9.4-1.1.4-.2.9 0 1.1.4l2.2 4.5c.2.4 0 .9-.4 1.1 0 .1-.1.1-.3.1zM39.6 46.6c-.3 0-.6-.2-.8-.5l-2.2-4.5c-.2-.4 0-.9.4-1.1.4-.2.9 0 1.1.4l2.2 4.5c.2.4 0 .9-.4 1.1-.1.1-.2.1-.3.1z"/></g></svg>');
}

.wi-snow {
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64"><g fill="white"><path d="M36.3 37.1h-9.1c-.2 0-.3-.1-.4-.3-.1-.2 0-.4.2-.5l7.3-4.8c.2-.1.4-.1.5.1.1.2 0 .4-.1.5l-5.9 3.9h7.6c.2 0 .4.2.4.4-.1.4-.3.7-.5.7zM36.3 42.7h-9.1c-.2 0-.3-.1-.4-.3-.1-.2 0-.4.2-.5l7.3-4.8c.2-.1.4-.1.5.1.1.2 0 .4-.1.5l-5.9 3.9h7.6c.2 0 .4.2.4.4 0 .4-.3.7-.5.7zM36.3 48.3h-9.1c-.2 0-.3-.1-.4-.3-.1-.2 0-.4.2-.5l7.3-4.8c.2-.1.4-.1.5.1.1.2 0 .4-.1.5l-5.9 3.9h7.6c.2 0 .4.2.4.4-.1.4-.3.7-.5.7z"/></g></svg>');
}

.wi-fog {
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64"><g fill="white"><path d="M23.7 41.6h16c.4 0 .8-.4.8-.8s-.4-.8-.8-.8h-16c-.4 0-.8.4-.8.8s.3.8.8.8zM23.7 37.3h16c.4 0 .8-.4.8-.8s-.4-.8-.8-.8h-16c-.4 0-.8.4-.8.8s.3.8.8.8zM23.7 45.9h16c.4 0 .8-.4.8-.8s-.4-.8-.8-.8h-16c-.4 0-.8.4-.8.8s.3.8.8.8z"/></g></svg>');
}

.wi-day-sunny {
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64"><circle fill="white" cx="32" cy="32" r="12"/><path fill="white" d="M32 20c.4 0 .8-.3.8-.8V16c0-.4-.3-.8-.8-.8s-.8.3-.8.8v3.2c0 .5.4.8.8.8zM32 44c-.4 0-.8.3-.8.8V48c0 .4.3.8.8.8s.8-.3.8-.8v-3.2c0-.5-.4-.8-.8-.8zM44 32c0-.4.3-.8.8-.8H48c.4 0 .8.3.8.8s-.3.8-.8.8h-3.2c-.5 0-.8-.4-.8-.8zM16 32c0 .4-.3.8-.8.8H12c-.4 0-.8-.3-.8-.8s.3-.8.8-.8h3.2c.5 0 .8.4.8.8zM40.8 23.2c.3-.3.8-.3 1.1 0l2.3 2.3c.3.3.3.8 0 1.1-.3.3-.8.3-1.1 0l-2.3-2.3c-.3-.3-.3-.8 0-1.1zM20.1 43.9c-.3.3-.8.3-1.1 0l-2.3-2.3c-.3-.3-.3-.8 0-1.1.3-.3.8-.3 1.1 0l2.3 2.3c.3.3.3.8 0 1.1zM40.8 40.8c.3.3.3.8 0 1.1l-2.3 2.3c-.3.3-.8.3-1.1 0-.3-.3-.3-.8 0-1.1l2.3-2.3c.3-.2.8-.2 1.1 0zM20.1 20.1c-.3-.3-.3-.8 0-1.1l2.3-2.3c.3-.3.8-.3 1.1 0 .3.3.3.8 0 1.1l-2.3 2.3c-.2.3-.7.3-1.1 0z"/></svg>');
}

.wi-day-cloudy {
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64"><path fill="white" d="M32 20c.4 0 .8-.3.8-.8V16c0-.4-.3-.8-.8-.8s-.8.3-.8.8v3.2c0 .5.4.8.8.8zM20.1 20.1c-.3-.3-.3-.8 0-1.1l2.3-2.3c.3-.3.8-.3 1.1 0 .3.3.3.8 0 1.1l-2.3 2.3c-.3.3-.8.3-1.1 0z"/><path fill="white" d="M25 22c0-1.7.4-3.3 1.1-4.7 1.9-3.7 5.7-6.3 10-6.3 5.5 0 10.1 4 11 9.2.3 0 .6 0 .9 0 4.4 0 8 3.6 8 8s-3.6 8-8 8H24c-4.4 0-8-3.6-8-8s3.6-8 8-8z"/></svg>');
}

.wi-cloudy {
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64"><path fill="white" d="M25 22c0-1.7.4-3.3 1.1-4.7 1.9-3.7 5.7-6.3 10-6.3 5.5 0 10.1 4 11 9.2.3 0 .6 0 .9 0 4.4 0 8 3.6 8 8s-3.6 8-8 8H24c-4.4 0-8-3.6-8-8s3.6-8 8-8z"/></svg>');
}

.weather-details {
  margin-left: 15px;
}

.temp {
  font-size: 36px;
  font-weight: bold;
  line-height: 1;
}

.description {
  font-size: 18px;
  opacity: 0.9;
  margin: 5px 0;
}

.feels-like {
  font-size: 14px;
  opacity: 0.8;
}

.weather-utility {
  background: rgba(255, 255, 255, 0.15);
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 15px;
}

.utility-section {
  margin-bottom: 15px;
}

.utility-section:last-child {
  margin-bottom: 0;
}

.utility-header {
  font-weight: 600;
  font-size: 16px;
  margin-bottom: 5px;
  opacity: 0.9;
}

.utility-content {
  font-size: 14px;
  opacity: 0.9;
}

.sun-time {
  display: flex;
  justify-content: space-between;
}

.sun-label {
  color: #888;
  margin-right: 8px;
}

.sun-value {
  font-weight: 500;
}

.mood-indicator {
  display: flex;
  align-items: center;
}

.mood-emoji {
  font-size: 28px;
  margin-right: 12px;
}

.mood-text {
  font-size: 16px;
  font-weight: 500;
}

.weather-info {
  display: flex;
  justify-content: space-between;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 8px;
  padding: 12px 15px;
  margin-bottom: 15px;
}

.info-item {
  text-align: center;
  flex: 1;
}

.info-label {
  font-size: 12px;
  opacity: 0.8;
  margin-bottom: 5px;
}

.info-value {
  font-size: 16px;
  font-weight: 600;
}

.update-time {
  text-align: right;
  font-size: 13px;
  opacity: 0.7;
}

@media (max-width: 480px) {
  .weather-container {
    padding: 15px;
  }

  .weather-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
  }
  
  .location-selector {
    width: 100%;
    flex-direction: column;
  }

  .location-input {
    width: 100%;
    margin-right: 0;
    margin-bottom: 10px;
  }

  .weather-main {
    flex-direction: column;
    align-items: center;
  }

  .weather-details {
    margin-top: 15px;
    padding-top: 15px;
    border-top: 1px solid rgba(0,0,0,0.1);
    border-left: none;
    padding-left: 0;
    width: 100%;
  }

  .detail-item {
    margin-right: 10px;
  }

  .weather-utility {
    flex-direction: column;
    gap: 15px;
  }

  .utility-section {
    width: 100%;
  }
}
</style> 