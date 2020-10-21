<template>
  <v-container fluid style="padding: 0">
    <mobile-header :title="pageTitle">
      <template>
        <v-btn icon large v-on:click="refresh()">
          <v-icon>refresh</v-icon>
        </v-btn>
      </template>
    </mobile-header>
    
    <v-card >
      <v-toolbar id="stations-toolbar" dense>
        <v-text-field id="searchKey" class="ml-2" clearable v-model="searchKey"></v-text-field>      
        
        <v-divider vertical class="mr-1"></v-divider>        
          
        <v-dialog v-model="filterMenu" persistent max-width="280px">
          <v-btn slot="activator" icon style="font-size:18px">
            <v-icon light>filter_list</v-icon>
          </v-btn>
          <v-card class="elevation-20">
            <v-card-text style="height: 300px; padding: 12px 0 0 0;">
              <v-layout row justify-space-between wrap >
                <v-flex xs12 sm12 pa-3>
                  <v-text-field
                    v-model="filter.id"
                    hide-details
                    label="油井ID"
                    type="number"
                    maxlength="16"
                  ></v-text-field>
                </v-flex>
                <v-flex xs12 sm12 pa-3>
                  <v-text-field
                    v-model="filter.title"
                    hide-details
                    label="油井名称"
                    type="number"
                    maxlength="16"
                  ></v-text-field>
                </v-flex>
                <v-flex xs12 sm12 pa-3>
                  <v-select
                      v-model="filter.warnTypes"
                      clearable
                      chips
                      small-chips
                      single-line
                      multiple
                      label="报警类型"
                      hide-details
                      :items="warnTypes"
                      item-value="id"
                      item-text="type"
                    >
                      <template slot="selection" slot-scope="{ item, index }">
                        <v-chip small v-if="index === 0">
                          <span>{{ item.type }}</span>
                        </v-chip>
                        <v-chip small v-if="index === 1">
                          <span>{{ item.type }}</span>
                        </v-chip>
                        <span
                          v-if="index === 2"
                          class="grey--text caption"
                        >+{{ filter.levels.length - 1 }}</span>
                      </template>
                    </v-select>
                </v-flex>
              </v-layout>
            </v-card-text>
            <v-spacer></v-spacer>
            <v-divider></v-divider>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn flat @click="filterCancelClick()">取消</v-btn>
              <v-btn color="primary" flat @click="filterApplyClick()">查询</v-btn>
            </v-card-actions>
          </v-card>
          <v-spacer></v-spacer>
        </v-dialog>        
      </v-toolbar>
        <v-layout row class="pt-3">
          <v-flex xs6 class="px-2">
            <v-menu
              v-model="datemenu"
              :close-on-content-click="false"
              :nudge-right="40"
              lazy
              transition="scale-transition"
              offset-y
              full-width
            >
              <template>
                <v-text-field
                  v-model="seldate"
                  label="请选择日期"
                  prepend-icon="event"
                  readonly
                  slot="activator"
                ></v-text-field>
              </template>
              <v-date-picker v-model="seldate" @input="datechanged"></v-date-picker>
            </v-menu>
          </v-flex>
        </v-layout>
      <template v-for="(data, index) in WarnData" >
          <v-card class="my-2"  v-bind:key="index">          
            <v-list dense class="px-3 py-2">
              <v-subheader>{{data.receivedAt}}</v-subheader>
              
              <v-list-tile>
                <v-list-tile-content>
                  <v-list-tile-title>站点编号</v-list-tile-title>
                  <v-list-tile-sub-title class="caption">
                  吉林油田001号
                  
                  </v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile>
              <v-divider></v-divider>
             
              <v-divider></v-divider>
              <v-list-tile>
                <v-list-tile-content>
                  <v-list-tile-title>报警类型</v-list-tile-title>
                  <v-list-tile-sub-title class="caption">
                    {{warns[data.alertType]}} 
                  </v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile>
              <v-divider></v-divider>
              <v-list-tile>
                <v-list-tile-content>
                  <v-list-tile-title>报警值</v-list-tile-title>
                  <v-list-tile-sub-title class="caption">
                    {{toNumValue(data.alertValue)}} 毫米
                  </v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile>
              <v-divider></v-divider>
              <v-list-tile>
                <v-list-tile-content>
                  <v-list-tile-title>报警预设值</v-list-tile-title>
                  <v-list-tile-sub-title class="caption">
                    {{toNumValue(data.setValue)}} 毫米
                  </v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile>
              <v-divider></v-divider>
              <!-- <v-list-tile>
                <v-list-tile-content>
                  <v-list-tile-title>位移四</v-list-tile-title>
                  <v-list-tile-sub-title class="caption">
                    {{toNumValue(data.move4)}} 毫米
                  </v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile> -->
              <v-divider></v-divider>
            
            </v-list>          
          </v-card>
        </template>

          
    </v-card>
  </v-container>
</template>

<script>
import moment from 'moment';
import api from '../modules/api.js';

export default {
  name: "Warns",
  data() {
    return {
      searchKey: "",
      filter: {},
      city: '白城',
      WarnData: [],
      refreshFlag: true,
      refreshInterval: 15,
      pageSize: 30,
      seldate: null,
      filterMenu: false,
      warns: [], 
       warnTypes: [],
      curWarn: null, 
      datemenu: false
    
    };
  },
  watch: {
    selChart: {
      handler() {
          let prevCStart=this.seldate+" 00:00:00";
          let prevCEnd=this.seldate+" 01:59:59";
          this.loadDataByDate(prevCStart,prevCEnd);    
      }
    }
  },
  computed: {
    hasMoreData: {
      get() {
        return true;
      }
    },
    pageTitle: {
      get() {
        return '报警 - ' + this.city;
      }
    }
  },
  methods: {
    getWarnTypeIcon(typeid) {
      switch(typeid) {
        case "倾角超标": 
          return "transit_enterexit";
        case "超温":
          return "whatshot";
        case "超湿":
          return "opacity";
        case "烟雾浓度超标":
          return "smoke_free";
        case "断电报警":
          return "battery_alert";
      }
      return "priority_high";
    },
    //filter menu
    filterCancelClick() {
      this.filterMenu = false;
    },
    filterApplyClick() {
      this.filterMenu = false;
    },

    //handle when click the station marker on map or in the list
    showDetail(warn) {
      this.$router.push({
        name: "warnDetail",
        params: { warn: warn }
      });
    },

    //reload stations
    refresh() {
      console.log("refresh");
      this.loadDataByDate();
      window.scrollTo(0, 0);
    },
    showFirstPage() {
      if (this.warns.length <= this.pageSize) {
        this.warns = this.warns;
      } else {
        this.warns = this.warns.slice(0, this.pageSize);
      }
    },
    showMore() {
      this.$utils.toast("没有更多");
    },
    datechanged() {
      this.datemenu = false;
      console.log(this.seldate);
      this.loadDataByDate();
    },
    toNumValue(v) {
      if (v && v != null && v != undefined) {
        return v.toFixed(2);
      }
      return '';
    },
    
    loadDataByDate() {
      if (this.seldate) {
        this.warnTypes=[];
        this.warns=[];
         this.$utils.showLoading();
         api.loadAlertType()
         .then(result => {
            this.$utils.hideLoading();
            this.WarnTypes = result;
            console.info(this.WarnTypes[1]);
           for (let i = 0;i<this.WarnTypes.length-1 ; i++){
                let r = this.WarnTypes[i];
               this.warns[i]=r.aName;
           }
          }).catch(err => {
            this.$utils.hideLoading();
            this.$utils.toast(`获取报警数据出错: ${err.message}`);
          })
       
            this.WarnData = [];
            this.$utils.showLoading();
            let start=this.seldate+" 00:00:00"; 
            let end=this.seldate+" 23:59:59";
            api.getWarnDataByDay(start,end)
            .then(result => {
              this.$utils.hideLoading();
              this.WarnData = result;
              console.info(this.WarnData);
             
            }).catch(err => {
              this.$utils.hideLoading();
              this.$utils.toast(`获取报警数据出错: ${err.message}`);
            })
        
      }
    }

  },
   beforeMount: function() { 
    this.seldate = new Date().toISOString().substr(0, 10);  
      // this.timer=setInterval(() => {
      //   this.nextHourChart();
      //   this.$utils.toast(selHourNum);
      // }, 1000); 
  },
  mounted: function() {
    
  },
  activated: function() {
    this.loadDataByDate();
  },
  deactivated: function() {}
};
</script>

<style scoped>
.search-box {
  position: absolute;
  top: 25px;
  left: 20px;
}
.container {
  padding-left: 0;
  padding-right: 0;
}
@media only screen and (max-width: 959px) {
  .container {
    padding-left: 0;
    padding-right: 0;
  }
}
#stations-toolbar >>> .v-toolbar__content {
  padding: 0px 8px 0px 8px !important;
}
.total_head {
  color: rgba(0, 0, 0, 0.54);
}
.stationItem >>> .v-list__tile {
  padding: 0px 8px 0px 6px !important;
}
.v-list__tile__avatar {
  min-width: 46px;
}
.v-list__tile__action {
  min-width: 16px;
}
i.hot_icon {
  font-size: 14px;
}
i.title_icon {
  font-size: 17px;
}
i.item_icon {
  font-size: 14px;
  color: rgba(0, 0, 0, 0.54);
  padding: 1px;
}
.amount_icon {
  font-size: 12px;
}
.totalArea {
  min-width: 54px;
  color: rgba(0, 0, 0, 0.54);
}
.cus-icon {
  position: relative;
  z-index: 1;
}
.cus-icon::before {
  content: "";
  position: absolute;
  z-index: -1;
  top: 0;
  left: 0;
  background-repeat: no-repeat;
}

#filter_toolbar_dialog {
  max-width: 290px;
}
.filter_toolbar_item {
  max-width: 270px;
}
.filter_toolbar_item >>> .v-list__tile {
  height: 40px !important;
}
.v-text-field--single-line >>> .v-input__slot {
  height: 36px;
  min-height: 36px;
}
.v-text-field--single-line >>> .v-input__control {
  min-height: 36px;
}
.v-text-field--single-line >>> input {
  margin-top: 0;
}
.filter_toolbar_item >>> .v-select__selections {
  flex-wrap: nowrap;
}
.map {
  width:100vw; 
  height: 78vh;
}
</style>
