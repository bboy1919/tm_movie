<template>
  <div class="city_body">
    <Scroller ref="city_sort">
      <div class="city_list">
        <div class="city_hot">
          <h2>热门城市</h2>
          <ul class="clearfix">
            <li
              v-for="hot in hotList"
              :key="hot.id"
              @tap="handleToCity(hot.nm, hot.id)"
            >{{ hot.nm }}</li>
          </ul>
        </div>
        <div class="city_sort" ref="citysort">
          <div v-for="city in cityList" :key="city.index">
            <h2>{{city.index}}</h2>
            <ul>
              <li v-for="cityitem in city.list" 
              :key="cityitem.id"
              @tap="handleToCity(cityitem.nm, cityitem.id)"
              >{{cityitem.nm}}</li>
            </ul>
          </div>
        </div>
      </div>
    </Scroller>
    <div class="city_index">
      <ul>
        <li
          v-for="(city, index) in cityList"
          :key="city.index"
          @touchstart="handleToIndex(index)"
        >{{city.index}}</li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: "City",
  data() {
    return {
      cityList: [],
      hotList: []
    };
  },
  mounted() {
    var cityList = window.localStorage.getItem("cityList");
    var hotCity = window.localStorage.getItem("hotCity");
    if (cityList && hotCity) {
      this.cityList = JSON.parse(cityList);
      this.hotList = JSON.parse(hotCity);

    } else {
      this.axios.get("/api/cityList").then(res => {
        var cities = res.data.data.cities;
        var { cityList, hotCity } = this.sCity(cities);
        this.hotList = hotCity;
        this.cityList = cityList;
        window.localStorage.setItem("hotCity", JSON.stringify(hotCity));
        window.localStorage.setItem("cityList", JSON.stringify(cityList));
      });
    }
  },
  methods: {
    sCity(cities) {
      var cityList = [];
      var hotCity = [];

      for (var i = 0; i < cities.length; i++) {
        if (cities[i].isHot === 1) {
          hotCity.push({ nm: cities[i].nm, id: cities[i].id });
        }
        var firstLetter = cities[i].py.substring(0, 1).toUpperCase();
        if (toCom(firstLetter)) {
          cityList.push({
            index: firstLetter,
            list: [{ nm: cities[i].nm, id: cities[i].id }]
          });
        } else {
          for (var j = 0; j < cityList.length; j++) {
            if (cityList[j].index === firstLetter) {
              cityList[j].list.push({ nm: cities[i].nm, id: cities[i].id });
            }
          }
        }
      }

      function toCom(firstLetter) {
        for (var i = 0; i < cityList.length; i++) {
          if (cityList[i].index === firstLetter) {
            return false;
          }
        }
        return true;
      }

      cityList.sort((n1, n2) => {
        if (n1.index > n2.index) {
          return 1;
        } else if (n1.index < n2.index) {
          return -1;
        } else {
          return 0;
        }
      });
      return {
        cityList,
        hotCity
      };
    },
    handleToIndex(index) {
      var h2 = this.$refs.citysort.getElementsByTagName("h2");
      // this.$refs.citysort.parentNode.scrollTop = h2[index].offsetTop;
      this.$refs.city_sort.toScrollTop(-h2[index].offsetTop);
    },
    handleToCity(nm, id){
      this.$store.commit('city/CITY_INFO', {nm, id });
      window.localStorage.setItem('nowNm', nm)
      window.localStorage.setItem('nowId', id)
      this.$router.push('/movie/nowPlaying');
    }
  }
};
</script>

<style scoped>
#content .city_body {
  margin-top: 45px;
  display: flex;
  width: 100%;
  position: absolute;
  top: 0;
  bottom: 0;
}
.city_body .city_list {
  flex: 1;
  overflow: auto;
  background: #fff5f0;
}
.city_body .city_list::-webkit-scrollbar {
  background-color: transparent;
  width: 0;
}
.city_body .city_hot {
  margin-top: 20px;
}
.city_body .city_hot h2 {
  padding-left: 15px;
  line-height: 30px;
  font-size: 14px;
  background: #f0f0f0;
  font-weight: normal;
}
.city_body .city_hot ul {
}
.city_body .city_hot ul li {
  float: left;
  background: #fff;
  width: 29%;
  height: 33px;
  margin-top: 15px;
  margin-left: 3%;
  padding: 0 4px;
  border: 1px solid #e6e6e6;
  border-radius: 3px;
  line-height: 33px;
  text-align: center;
  box-sizing: border-box;
}
.city_body .city_sort {
}
.city_body .city_sort div {
  margin-top: 20px;
}
.city_body .city_sort h2 {
  padding-left: 15px;
  line-height: 30px;
  font-size: 14px;
  background: #f0f0f0;
  font-weight: normal;
}
.city_body .city_sort ul {
  padding-left: 10px;
  margin-top: 10px;
}
.city_body .city_sort ul li {
  line-height: 30px;
  line-height: 30px;
}
.city_body .city_index {
  width: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  text-align: center;
  border-left: 1px #e6e6e6 solid;
}
</style>
