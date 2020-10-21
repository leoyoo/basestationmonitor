<template>
  <v-container fluid>
    <mobile-h title="油田护航者">
      <template slot="left">
       
      </template>
    </mobile-h>
    <mobile-header title="油井实时监测数据">
      <template slot="left">
        <v-btn icon large v-on:click="goBack()">
          <v-icon medium>keyboard_arrow_left</v-icon>
        </v-btn>        
      </template>
      <template>
        <v-btn icon large @click="refresh()">
          <v-icon>refresh</v-icon>
        </v-btn>
      </template>
    </mobile-header>

    <!-- station info  -->
    <v-card  class="mb-1 py-2 pb-4">
      <v-layout align-center >
        <v-flex xs10 class="px-3 py-1">          
                <span class="">
                  <v-icon small>perm_device_information</v-icon>
                  {{ station.name }} [{{ station.tag }}]
                </span>
        </v-flex>
        <v-flex xs2 class="pr-5 py-2">
          <v-btn icon small @click="editstation">
            <!-- <v-icon small>edit</v-icon> -->
            <v-badge>
              <v-img :src="edit_icon" width="20px" height="20px"></v-img>
            </v-badge>
          </v-btn>        
        </v-flex>        
      </v-layout>
      <v-layout align-center justify-end row class="px-3">
        <v-flex xs12>
          <span class="">
            <v-icon small>place</v-icon>
              {{station.city}}, {{station.province}}
          </span>
        </v-flex>
      </v-layout>
    </v-card>

    <v-tabs
        slot="extension"
        v-model="seltab"
        fixed-tabs
        color=""
        v-if="!error"
        @change="tabchange"
      >
      <v-tabs-slider></v-tabs-slider>
      <v-tab href="#basic" class="primary--text">
        <!-- <v-icon>description</v-icon> -->
        <v-badge>
          <v-img :src="data_icon" width="20px" height="20px"></v-img>
        </v-badge>
      </v-tab>

      <v-tab href="#daily" class="primary--text"  >
        <!-- <v-icon>list_alt</v-icon> -->
        <v-badge>
          <v-img :src="daily_icon" width="20px" height="20px"></v-img>
        </v-badge>
      </v-tab>

      <v-tab href="#chart" class="primary--text">
        <!-- <v-icon>insert_chart_outlined</v-icon> -->
        <v-badge>
          <v-img :src="chart_icon" width="20px" height="20px"></v-img>
        </v-badge>
      </v-tab>

      <v-tab href="#comments" class="primary--text">
        <!-- <v-icon>comment</v-icon> -->
        <v-badge>
          <v-img :src="comment_icon" width="20px" height="20px"></v-img>
        </v-badge>
      </v-tab>
    </v-tabs>
   
    <v-tabs-items v-model="seltab" class="white elevation-1" v-if="!error">
      <v-tab-item id="basic">
        
        <!-- Coordinators -->
        <template v-for="(coor, index) in coordinators">
          <v-card v-if="!error" class="mb-2 pt-2" v-bind:key="index">
            <v-layout align-center>
              <v-flex>
                <v-list dense class="pl-3 pr-3">
                  <v-subheader>协调器{{coor.seqId}} - {{coor.address}}</v-subheader>
                  <v-list-tile>
                    <v-list-tile-content>
                      <v-list-tile-title>水平倾斜角度</v-list-tile-title>
                      <v-list-tile-sub-title class="caption">
                        X轴: {{toNumValue(coor.data.tilt1X)}} 度
                        Y轴: {{toNumValue(coor.data.tilt1Y)}} 度
                      </v-list-tile-sub-title>
                    </v-list-tile-content>
                  </v-list-tile>
                  <v-divider></v-divider>
                  <!-- <v-list-tile>
                    <v-list-tile-content>
                      <v-list-tile-title>垂直偏移距离</v-list-tile-title>
                      <v-list-tile-sub-title class="caption">
                        X轴: {{toNumValue(coor.data.skewingX)}} 毫米
                        Y轴: {{toNumValue(coor.data.skewingY)}} 毫米 
                      </v-list-tile-sub-title>
                    </v-list-tile-content>
                  </v-list-tile> -->
                  <v-divider></v-divider>
                  <v-list-tile>
                    <v-list-tile-content>
                      <v-list-tile-title>左侧位移</v-list-tile-title>
                      <v-list-tile-sub-title class="caption">
                        {{toNumValue(coor.data.move1)}} 毫米
                      </v-list-tile-sub-title>
                    </v-list-tile-content>
                  </v-list-tile>
                  <v-divider></v-divider>
                  <v-list-tile>
                    <v-list-tile-content>
                      <v-list-tile-title>左前侧位移</v-list-tile-title>
                      <v-list-tile-sub-title class="caption">
                        {{toNumValue(coor.data.move2)}} 毫米
                      </v-list-tile-sub-title>
                    </v-list-tile-content>
                  </v-list-tile>
                  <v-divider></v-divider>
                   <v-list-tile>
                    <v-list-tile-content>
                      <v-list-tile-title>右前侧位移</v-list-tile-title>
                      <v-list-tile-sub-title class="caption">
                        {{toNumValue(coor.data.move3)}} 毫米
                      </v-list-tile-sub-title>
                    </v-list-tile-content>
                  </v-list-tile>
                  <v-divider></v-divider> 
                  <v-list-tile>
                    <v-list-tile-content>
                      <v-list-tile-title>右侧位移</v-list-tile-title>
                      <v-list-tile-sub-title class="caption">
                        {{toNumValue(coor.data.move4)}} 毫米
                      </v-list-tile-sub-title>
                    </v-list-tile-content>
                  </v-list-tile>
                  <v-divider></v-divider>

                  <!-- <v-list-tile>
                    <v-list-tile-content>
                      <v-list-tile-title>倾角测量仪2 {{coor.data? coor.data.tilt2_Addr : ''}}</v-list-tile-title>
                      <v-list-tile-sub-title class="caption">
                        X轴: {{coor.data? `${coor.data.tilt2_X_Positive > 0? '+' : '-'}${coor.data.tilt2_X_Degree}°${coor.data.tilt2_X_Minute}'${coor.data.tilt2_X_Second}"` : ''}} 
                        Y轴: {{coor.data? `${coor.data.tilt2_Y_Positive > 0? '+' : '-'}${coor.data.tilt2_Y_Degree}°${coor.data.tilt2_Y_Minute}'${coor.data.tilt2_Y_Second}"` : ''}} 
                      </v-list-tile-sub-title>
                    </v-list-tile-content>
                  </v-list-tile>
                  <v-divider></v-divider>
                  <v-list-tile>
                    <v-list-tile-content>
                      <v-list-tile-title>倾角测量仪3 {{coor.data? coor.data.tilt3_Addr : ''}}</v-list-tile-title>
                      <v-list-tile-sub-title class="caption">
                        X轴: {{coor.data? `${coor.data.tilt3_X_Positive > 0? '+' : '-'}${coor.data.tilt3_X_Degree}°${coor.data.tilt3_X_Minute}'${coor.data.tilt3_X_Second}"` : ''}} 
                        Y轴: {{coor.data? `${coor.data.tilt3_Y_Positive > 0? '+' : '-'}${coor.data.tilt3_Y_Degree}°${coor.data.tilt3_Y_Minute}'${coor.data.tilt3_Y_Second}"` : ''}} 
                      </v-list-tile-sub-title>
                    </v-list-tile-content>
                  </v-list-tile>
                  <v-divider></v-divider>
                  <v-list-tile>
                    <v-list-tile-content>
                      <v-list-tile-title>倾角测量仪4 {{coor.data? coor.data.tilt4_Addr : ''}}</v-list-tile-title>
                      <v-list-tile-sub-title class="caption">
                        X轴: {{coor.data? `${coor.data.tilt4_X_Positive > 0? '+' : '-'}${coor.data.tilt4_X_Degree}°${coor.data.tilt4_X_Minute}'${coor.data.tilt4_X_Second}"` : ''}} 
                        Y轴: {{coor.data? `${coor.data.tilt4_Y_Positive > 0? '+' : '-'}${coor.data.tilt4_Y_Degree}°${coor.data.tilt4_Y_Minute}'${coor.data.tilt4_Y_Second}"` : ''}} 
                      </v-list-tile-sub-title>
                    </v-list-tile-content>
                  </v-list-tile>
                  <v-divider></v-divider> -->
                  <v-list-tile>
                    <v-list-tile-content>最后数据时间</v-list-tile-content>
                    <v-list-tile-content class="caption">
                      {{coor.data? `${coor.data.receivedAt}` : ''}}
                    </v-list-tile-content>
                  </v-list-tile>  
                </v-list>
              </v-flex>
            </v-layout>
          </v-card>
        </template>
        
        
        <!-- action buttons -->
        <v-card v-if="!error" class="mb-2">
          <v-layout align-center justify-center>
            <v-flex>
              <div class="text-xs-center">
                <v-btn color="#4A87D3" dark @click.native="gotoMap">
                  查看地图
                  <v-icon right dark>map</v-icon>
                </v-btn>
                <!-- <v-btn color="#4A87D3" dark @click.native="gotoWarns">
                  查看报警记录
                  <v-icon right dark>warning</v-icon>
                </v-btn>           -->
              </div>
            </v-flex>
          </v-layout>
          </v-card>
      </v-tab-item>
      <v-tab-item id="daily">
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
          <v-flex xs6 class="px-2">
            <v-select
              :items="coordinators"
              :item-text="coorlabel"
              return-object
              label="选择协调器"
              v-model="selcoor"
              @change="coorchanged"
            ></v-select>
          </v-flex>
        </v-layout>
        <v-layout row >
          <v-flex xs5 class="px-2">
            <v-switch v-model="showDailyChart" label="显示曲线"></v-switch>
          </v-flex>
          <v-flex xs7 class="px-2">
            <v-select :items="chartOptions" item-text="text" item-value="value" v-model="selChart" v-if="showDailyChart"></v-select>
          </v-flex>
          </v-layout>
           <v-layout row >
           <v-flex xs10 class="px-2"> 
            <v-icon  @click="prevHourChart"  v-if="showDailyChart">arrow_back</v-icon>
          </v-flex>
          <v-list-tile-sub-title >
                        第{{this.selHourNum}} 页，共12页
                      </v-list-tile-sub-title>
          
           <v-flex  xs2 class="px-2">
            <v-icon  @click="nextHourChart"  v-if="showDailyChart">arrow_forward</v-icon>
          </v-flex>
        </v-layout>

        <template v-for="(data, index) in dailyData" v-if="!showDailyChart">
          <v-card class="my-2"  v-bind:key="index">          
            <v-list dense class="px-3 py-2">
              <v-subheader>{{data.receivedAt}}</v-subheader>
              
              <v-list-tile>
                <v-list-tile-content>
                  <v-list-tile-title>水平倾斜角度</v-list-tile-title>
                  <v-list-tile-sub-title class="caption">
                    X轴: {{toNumValue(data.tilt1X)}} 度
                    Y轴: {{toNumValue(data.tilt1Y)}} 度
                  </v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile>
              <v-divider></v-divider>
              <!-- <v-list-tile>
                <v-list-tile-content>
                  <v-list-tile-title>垂直偏移距离</v-list-tile-title>
                  <v-list-tile-sub-title class="caption">
                    X轴: {{toNumValue(data.skewingX)}} 毫米
                    Y轴: {{toNumValue(data.skewingY)}} 毫米 
                  </v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile> -->
              <v-divider></v-divider>
              <v-list-tile>
                <v-list-tile-content>
                  <v-list-tile-title>左侧位移</v-list-tile-title>
                  <v-list-tile-sub-title class="caption">
                    {{toNumValue(data.move11)}} 毫米
                  </v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile>
              <v-divider></v-divider>
              <v-list-tile>
                <v-list-tile-content>
                  <v-list-tile-title>左前侧位移</v-list-tile-title>
                  <v-list-tile-sub-title class="caption">
                    {{toNumValue(data.move12)}} 毫米
                  </v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile>
              <v-divider></v-divider>
              <v-list-tile>
                <v-list-tile-content>
                  <v-list-tile-title>右前侧位移</v-list-tile-title>
                  <v-list-tile-sub-title class="caption">
                    {{toNumValue(data.move13)}} 毫米
                  </v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile>
              <v-divider></v-divider>
              <v-list-tile>
                <v-list-tile-content>
                  <v-list-tile-title>右侧位移</v-list-tile-title>
                  <v-list-tile-sub-title class="caption">
                    {{toNumValue(data.move14)}} 毫米
                  </v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile>
              <v-divider></v-divider>
              <!-- <v-list-tile>
                <v-list-tile-content>
                  <v-list-tile-title>倾角测量仪2 {{data.tilt2_Addr}}</v-list-tile-title>
                  <v-list-tile-sub-title class="caption">
                    X轴: {{`${data.tilt2_X_Positive > 0? '+' : '-'}${data.tilt2_X_Degree}°${data.tilt2_X_Minute}'${data.tilt2_X_Second}"`}} 
                    Y轴: {{`${data.tilt2_Y_Positive > 0? '+' : '-'}${data.tilt2_Y_Degree}°${data.tilt2_Y_Minute}'${data.tilt2_Y_Second}"`}} 
                  </v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile>
              <v-divider></v-divider>
              <v-list-tile>
                <v-list-tile-content>
                  <v-list-tile-title>倾角测量仪3 {{data.tilt3_Addr}}</v-list-tile-title>
                  <v-list-tile-sub-title class="caption">
                    X轴: {{`${data.tilt3_X_Positive > 0? '+' : '-'}${data.tilt3_X_Degree}°${data.tilt3_X_Minute}'${data.tilt3_X_Second}"`}} 
                    Y轴: {{`${data.tilt3_Y_Positive > 0? '+' : '-'}${data.tilt3_Y_Degree}°${data.tilt3_Y_Minute}'${data.tilt3_Y_Second}"`}} 
                  </v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile>
              <v-divider></v-divider>
              <v-list-tile>
                <v-list-tile-content>
                  <v-list-tile-title>倾角测量仪4 {{data.tilt4_Addr}}</v-list-tile-title>
                  <v-list-tile-sub-title class="caption">
                    X轴: {{`${data.tilt4_X_Positive > 0? '+' : '-'}${data.tilt4_X_Degree}°${data.tilt4_X_Minute}'${data.tilt4_X_Second}"`}} 
                    Y轴: {{`${data.tilt4_Y_Positive > 0? '+' : '-'}${data.tilt4_Y_Degree}°${data.tilt4_Y_Minute}'${data.tilt4_Y_Second}"`}} 
                  </v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile>   -->
            </v-list>          
          </v-card>
        </template>

        <v-card class="pa-2" v-if="showDailyChart">
            <!-- <line-chart :chart-data="stationDatas" :options="chartOption"></line-chart> -->
          <div v-if="selChart <6" class="chart-container" style="position: relative; height:350px; width:90vw">
            <canvas id="candailyChart"></canvas>
          </div>
        </v-card>

        <v-btn absolute icon class="upbtn" color="#4A87D3" @click="uptop">
          <v-icon>arrow_upward</v-icon>
        </v-btn>
      </v-tab-item>
      <v-tab-item id="chart">
        <v-layout row class="pt-3">
          <v-flex xs6 class="px-2">
            <v-menu
              v-model="startdatemenu"
              :close-on-content-click="false"
              :nudge-right="40"
              lazy
              transition="scale-transition"
              offset-y
              full-width
            >
              <template>
                <v-text-field
                  v-model="selstartdate"
                  label="开始日期"
                  prepend-icon="event"
                  readonly
                  slot="activator"
                ></v-text-field>
              </template>
              <v-date-picker v-model="selstartdate" @input="startdatemenu = false"></v-date-picker>
            </v-menu>
          </v-flex>
          <v-flex xs6 class="px-2">
            <v-menu
              v-model="enddatemenu"
              :close-on-content-click="false"
              :nudge-right="40"
              lazy
              transition="scale-transition"
              offset-y
              full-width
            >
              <template>
                <v-text-field
                  v-model="selenddate"
                  label="结束日期"
                  prepend-icon="event"
                  readonly
                  slot="activator"
                ></v-text-field>
              </template>
              <v-date-picker v-model="selenddate" @input="enddatemenu = false"></v-date-picker>
            </v-menu>
          </v-flex>
        </v-layout>
        <v-layout row class="pb-2">
          <v-flex xs7 class="px-4 py-0">
            <v-select :items="chartOptions" item-text="text" item-value="value" v-model="selChart"></v-select>
          </v-flex>
          <v-flex xs5 class="px-2 pt-2">
            <v-btn small @click="refreshChartData">更新数据</v-btn>
          </v-flex>
        </v-layout>

        <v-card class="pa-2">
            <!-- <line-chart :chart-data="stationDatas" :options="chartOption"></line-chart> -->
          <div v-if="selChart <6" class="chart-container" style="position: relative; height:350px; width:90vw">
            <canvas id="dataChart"></canvas>
          </div>
        </v-card>
      </v-tab-item>
      <v-tab-item id="comments">
        <v-card v-if="!error" class="mb-2">
          <v-list dense two-line>            
            <v-list-tile>
              <v-list-tile-content>                
              </v-list-tile-content>
              <v-list-tile-action>
                <v-btn icon @click="showAddComment=true">
                  <!-- <v-icon dark>add</v-icon> -->
                  <v-badge>
                    <v-img :src="comment_add_icon" width="20px" height="20px"></v-img>
                  </v-badge>
                </v-btn>
              </v-list-tile-action>
            </v-list-tile>
          </v-list>
          <v-list three-line dense>
            <v-subheader>
                备注记录
            </v-subheader>            
            <template v-for="(comment, index) in comments">
              <v-divider
                :key="index"
              ></v-divider>
              <v-list-tile
                :key="comment.comment"
                avatar
              >
                <v-list-tile-content>
                  <v-list-tile-title><v-icon small>speaker_notes</v-icon>&nbsp;{{comment.comment}}</v-list-tile-title>
                  <v-list-tile-sub-title class="caption">{{comment.commentAt}}</v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile>
            </template>
          </v-list>
        </v-card>
      </v-tab-item>
    </v-tabs-items>

    <v-card v-if="error" height="100%">
      <v-card-title primary-title>
        <div>
          <h3 class="headline mb-0">
            <v-icon large color="red darken-2">error_outline</v-icon>
            {{error}}
          </h3>
        </div>
      </v-card-title>
      <v-card-text>{{this.$t("playerDetail.errorAction")}}</v-card-text>
    </v-card>
    
    <edit-station :show-edit-dialog="showEditDialog"
          :station="station"
          v-on:save="onEditStationSave" v-on:cancel="onEditStationCancel">
    </edit-station> 

    <v-dialog v-model="showAddComment" persistent max-width="85vw" style="z-index: 899">
      <v-card class="elevation-20 ">
        <v-card-title
            primary-title
          >
            添加备注
            <v-spacer></v-spacer>
        </v-card-title>
        <v-card-text style="">        
          <v-list two-line subheader dense>
            <v-list-tile class="filter_toolbar_item">
              <v-textarea rows="2" name="commentArea" placeholder="增加备注" auto-grow v-model="newComment" class="caption"></v-textarea>              
            </v-list-tile>
          </v-list>
        </v-card-text>
        <v-divider></v-divider>   
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn flat @click="showAddComment=false">取消</v-btn>
          <v-btn color="primary" flat @click="addComment">确定</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

  </v-container>
</template>

<script>

import Chart from 'chart.js';
import api from '../modules/api.js';
import moment, { min } from 'moment';
import EditStation from '../components/EditStation.vue';
//var this.selHourNum=0;
var minData=0;
var maxData=1;

export default {
  name: "StationDetail",
  props: ['station'],
  data() {
    return {
      error: null,
      newComment: null,
      comments: [],
      asts: [],
      seltab: 'basic',
      showDailyChart: true,
      coordinators: [],
      dailyData: [],
      dataChart: null,
      dailyChart: null,
      labelList:['倾角X轴(度)','倾角Y轴(度)','左侧位移(毫米)','左前侧位移(毫米)','右前侧位移(毫米)','右侧位移(毫米)'],
     
      chartOptions: [
        { text: '水平倾角X轴', value: 0 },
        { text: '水平倾角Y轴', value: 1 },     
        { text: '左侧位移', value: 2 },
        { text: '左前侧位移', value: 3 },
        { text: '右前侧位移', value: 4 },
        { text: '右侧位移', value: 5 }
      ],
      selChart: 0,
      selHourNum:0,
      chartOption: {

        responsive: true, 
        hover: { mode: 'nearest', intersect: true },
        showLine: false, 
        spanGaps: true, 
        steppedLine: true,
        scales: {
          xAxes: [{display: true, scaleLabel: {display: true, labelString: '日期' }}], 
          yAxes: [{display: true,scaleLabel: {display: true, labelString: '倾角(度)'}}]
        }
      },
      stationDatas: {
        labels: [],
        datasets: [{
            label: '倾角X轴(度)', 
            backgroundColor: '#4DD0E1',
            borderColor: '#4DD0E1',
            data: [],
            fill: false
          }, {
            label: '倾角Y轴(度)', 
            backgroundColor: '#CDDC39',
            borderColor: '#CDDC39',
            data: [],
            fill: false
          }]          
      },
      dailyDatas: {
        labels: [],
        datasets: [{
            label: 'X轴(度)', 
            backgroundColor: '#4DD0E1',
            borderColor: '#4DD0E1',
            data: [],
            fill: false
          }, {
            label: 'Y轴(度)', 
            backgroundColor: '#CDDC39',
            borderColor: '#CDDC39',
            data: [],
            fill: false
          }]
      },
      datemenu: false,
      seldate: null,
      selcoor: null,
      showEditDialog: false,
      startdatemenu: false,
      enddatemenu: false,
      selstartdate: null,
      selenddate: null,
      showAddComment: false
      
    };
  },
  components: {
    EditStation
  },
  watch: {
    selChart: {
      handler() {
        if (this.seltab === 'daily') {
          let prevCStart=this.seldate+" 00:00:00";
          let prevCEnd=this.seldate+" 01:59:59";
          this.refreshDailyChartData(prevCStart,prevCEnd);
        } else if (this.seltab === 'chart') {
          this.refreshChartData();
        }        
      }
    },
    seldate: {
      handler() {
        this.refresh();
      }
    },
    showDailyChart: {
      handler() {
        this.refresh();
      }      
    }
  },
  computed: {
    data_icon: {
      get() {
        return this.seltab === 'basic'? this.$utils.getImageUrl('data.png') : this.$utils.getImageUrl('data_disabled.png') ;
      }
    },
    daily_icon: {
      get() {
        return this.seltab === 'daily'? this.$utils.getImageUrl('coordinator.png') : this.$utils.getImageUrl('coordinator_disabled.png');
      }
    },
    chart_icon: {
      get() {
        return this.seltab === 'chart'? this.$utils.getImageUrl('calendar.png') : this.$utils.getImageUrl('calendar_disabled.png');
      }
    },
    comment_icon: {
      get() {
        return this.seltab === 'comments'? this.$utils.getImageUrl('comment.png') : this.$utils.getImageUrl('comment_disabled.png');
      }
    },
    map_icon: {
      get() {
        return this.$utils.getImageUrl('map.png');
      }
    },
    edit_icon: {
      get() {
        return this.$utils.getImageUrl('edit_st.png');
      }
    },
    comment_add_icon: {
      get() {
        return this.$utils.getImageUrl('comment_add.png');
      }
    },
    left_icon:  {
       get() {
        return this.$utils.getImageUrl('left.png');
      }
    },
    right_icon:  {
       get() {
        return this.$utils.getImageUrl('right.png');
      }
    }
  },
  methods: {
    goBack() {
      this.$router.go(-1);
    },    
    gotoMap() {
      if (this.station.lng && this.station.lat) {
        this.$router.push({
          name: "stations",
          params: { displayStationId: this.station.id }
        });
      } else {
        this.$utils.toast(`这个油井还没有设置地理位置`);
      }
    },
    refresh() {
      if (this.seltab === 'basic') {
        this.loadCoordinators();
      } else if (this.seltab === 'daily') {
        if (this.showDailyChart) {
          let prevCStart=this.seldate+" 00:00:00";
          let prevCEnd=this.seldate+" 01:59:59";
          this.refreshDailyChartData(prevCStart,prevCEnd);
        } else {
          this.loadDataByDate();
        }        
      } else if (this.seltab === 'comments') {
        this.loadComments();
      } else if (this.seltab === 'chart') {
        this.refreshChartData();
      }      
    },
    gotoWarns() {
      this.$router.push({
        name: "warnsHistory",
        params: { stationId: this.station.id }
      });
    },
    addComment() {
      if (this.newComment) {
        this.showAddComment = false;
        api.addComment(this.station.id, this.newComment)
        .then(() => {
          this.$utils.toast(`添加备注成功`);
          this.newComment = null;
          this.loadComments();
        }).catch(err => {
          this.$utils.toast(`添加备注出错: ${err.message}`);
        });
      }
    },
    loadCoordinators() {
      api.getStationCoordinators(this.station.id)
      .then(result => {
        this.coordinators = result;
        if (this.coordinators && this.coordinators.length > 0) {
          this.selcoor = this.coordinators[0];
        }
        this.loadLatestData();
      }).catch(err => {
        this.$utils.toast(`获取协调器信息失败: ${err.message}`);
      })
    },
    loadLatestData() {
      if (this.coordinators && this.coordinators.length > 0) {
        for (let i = 0; i < this.coordinators.length; i++) {
          let coor = this.coordinators[i];
          api.getStationDataLatest(coor.stationId, coor.seqId)
          .then(data => {
            coor.data = data;
            this.$set(this.coordinators, i, coor);
          }).catch(err => {
            this.$utils.toast(`获取最新数据失败: ${err.message}`);
          });
        }
      }
    },
    loadDataByDate() {
      if (this.seldate && this.selcoor) {
        this.dailyData = [];
        this.$utils.showLoading();
        let start=this.seldate+" 00:00:00"; 
        let end=this.seldate+" 23:59:59";
        api.getStationDataByDay(this.selcoor.stationId, this.selcoor.seqId, start,end)
        .then(result => {
          this.$utils.hideLoading();
          this.dailyData = result;
        }).catch(err => {
          this.$utils.hideLoading();
          this.$utils.toast(`获取历史数据出错: ${err.message}`);
        })
      }
    },
    loadComments() {
      this.comments = [];
      this.$utils.showLoading();
      api.getStationComments(this.station.id)
      .then(result => {
        this.$utils.hideLoading();
        this.comments = result
      }).catch(err => {
        this.$utils.hideLoading();
        this.$utils.toast(`获取油井备注信息出错: ${err.message}`);
      })
    },
    // loadAlertSetting() {
      // this.asts = [];
      //  this.$utils.showLoading();
      // api.loadAlertSetting()
      // .then(result => {
      //   this.$utils.hideLoading();
      //   this.asts = result;
      //   console.info(this.asts);
      // }).catch(err => {
      //   this.$utils.hideLoading();
      //   this.$utils.toast(`获取报警设置信息出错: ${err.message}`);
      // })
    // },
    tabchange(value) {
      console.log(`Tab selected: ${value}`);
      if (this.seltab === 'comments') {
        this.loadComments();
      }
    },
    datechanged() {
      this.datemenu = false;
      console.log(this.seldate);
      this.loadDataByDate();
    },
    coorchanged() {
      console.log(this.selcoor);
      this.loadDataByDate();
    },
    coorlabel(item) {
      return `${item.seqId} - ${item.address}`;
    },
    uptop() {
      document.body.scrollTop = 0; // For Safari
      document.documentElement.scrollTop = 0; // For Chrome, Firefox, IE and Opera
    },
    editstation() {
      this.showEditDialog = true;
    },
    onEditStationSave() {
      this.showEditDialog = false;
    },
    onEditStationCancel() {
      this.showEditDialog = false;
    },
   
    drawChartData() {
      // let cxt = document.getElementById('dataChart'+this.selChart).getContext('2d');
       if(this.dataChart!=null){
        this.dataChart.destroy();
      }
     
      let canvas=document.getElementById('dataChart');
      let cxt = canvas.getContext('2d');
      let label = this.labelList[this.selChart];
      let step = 1;
     
      this.dataChart = new Chart(cxt, {
        type: 'line',
        data: this.stationDatas,
        options: {
          responsive: true, 
          maintainAspectRatio: false,
            legend: {
                display:true,
                position: 'top', // 显示的位置
                fullWidth: 'true',     
                  labels:{ //图例标签配置
                        boxWidth:10 ,//彩色框的宽度
                  
                }        
            },
          tooltips: {
            callbacks: {
            label: function (tooptipItem) {
              return tooptipItem.yLabel ;
            }
            }
          },
          hover: { mode: 'nearest', intersect: true ,animationDuration: 0},
          animation: {           // 这部分是数值显示的功能实现
                onComplete: function () {
                    var chartInstance = this.chart,
 
                    ctx = chartInstance.ctx;
                     // 以下属于canvas的属性（font、fillStyle、textAlign...）
                    ctx.font = Chart.helpers.fontString(Chart.defaults.global.defaultFontSize, Chart.defaults.global.defaultFontStyle, Chart.defaults.global.defaultFontFamily);
                    ctx.fillStyle = "black";
                    ctx.textAlign = 'center';
                    ctx.textBaseline = 'bottom';
 
                    this.data.datasets.forEach(function (dataset, i) {
                      if(i==0 ){
                        var meta = chartInstance.controller.getDatasetMeta(i);
                        meta.data.forEach(function (bar, index) {
                            var data = dataset.data[index];
                            ctx.fillText(data, bar._model.x, bar._model.y - 5);
                        });
                      }
                    });
                }
          },
          showLine: true, 
          spanGaps: true, 
          steppedLine: true,
          scales: {
            xAxes: [{display: true, scaleLabel: {display: true, labelString: '日期' }}], 
            yAxes: [{display: true, scaleLabel: {display: true, labelString: label}, ticks:{min:Math.round(minData),max:Math.round(maxData)}}]
          } 
        }
      });
      minData=0;
      maxData=1;
      
    },
   
    drawDailyChart() {
      if(this.dailyChart!=null){
        this.dailyChart.destroy();
      }
      
      let canvas=document.getElementById('candailyChart');
      let cxt = canvas.getContext('2d');
     
      let step = 1;
    
      let label=this.labelList[this.selChart];
         
        this.dailyChart = new Chart(cxt, {
        type: 'line',
        data: this.dailyDatas,
       
        options: {
          
          responsive: true, 
          maintainAspectRatio: false,
          legend: {
                display:true,
                position: 'top', // 显示的位置
                fullWidth: 'true',     
                  labels:{ //图例标签配置
                        boxWidth:10 ,//彩色框的宽度
                  
                }        
            },
          tooltips: {
            callbacks: {
            label: function (tooptipItem) {
              return tooptipItem.yLabel ;
            }
            }
          },
          hover: { mode: 'nearest', intersect: true ,animationDuration: 0},
          animation: {           // 这部分是数值显示的功能实现
                onComplete: function () {
                    var chartInstance = this.chart,
 
                    ctx = chartInstance.ctx;             
                    // 以下属于canvas的属性（font、fillStyle、textAlign...）
                    ctx.font = Chart.helpers.fontString(Chart.defaults.global.defaultFontSize, Chart.defaults.global.defaultFontStyle, Chart.defaults.global.defaultFontFamily);
                    ctx.fillStyle = "black";
                    ctx.textAlign = 'center';
                    ctx.textBaseline = 'bottom';
                    this.data.datasets.forEach(function (dataset, i) {
                      if(i==0 ){
                        var meta = chartInstance.controller.getDatasetMeta(i);
                        meta.data.forEach(function (bar, index) {
                            var data = dataset.data[index];                        
                            ctx.fillText(data, bar._model.x, bar._model.y - 5);  
                                                     
                        });
                      }
                    });
                    
                }
          },
          showLine: true, 
          spanGaps: true, 
          steppedLine: true,
          
          scales: {
            xAxes: [{display: true, scaleLabel: {display: true, labelString: '时间' }}], 
            yAxes: [{display: true, scaleLabel: {display: true, labelString: label},ticks:{min:Math.round(minData-0.5),max:Math.round(maxData+0.5)}}]
          } 
          
        }
      });
      minData=0;
      maxData=1;
     
    },

    refreshChartData() {
      this.stationDatas = null;
      let displayData=[];
          this.stationDatas = {
          labels: [],
          datasets: [{
              label: this.labelList[this.selChart],
              backgroundColor: '#4DD0E1',
              borderColor: '#4DD0E1',
              pointBorderWidth: 4,
              data: [],
              fill: false
            }, {
            label: '报警', 
            backgroundColor: 'red',
            borderColor: 'red',
            borderDash: [10,3],
            radius:"0",
            borderWidth : 1,//线条的宽度
            data: [],
            fill: false
          }, {
            label:"",
            backgroundColor: 'red',
            borderColor: 'red',
            borderDash: [10,3],
            borderWidth : 1,//线条的宽度
            radius:"0",
            data: [],
            fill: false
          }, {
            label: '预警', 
            backgroundColor: '#FF7F24',
            borderColor: '#FF7F24',
            borderDash: [10,3],
            borderWidth : 1,//线条的宽度
            radius:"0",
            data: [],
            fill: false
          },{
            label:"",
            backgroundColor: '#FF7F24',
            borderColor: '#FF7F24',
            borderDash: [10,3],
            borderWidth : 1,//线条的宽度
            radius:"0",
            data: [],
            fill: false
          }]          
          };
        this.$utils.showLoading();
      api.loadAlertSetting()
      .then(result => {
        this.$utils.hideLoading();
        this.asts = result;
        let r = this.asts[0];  
        let WarnValue=[0,0,0,0];
        let xAlert=[r.tiltXMax,r.tiltXMin,r.tiltXPreMax,r.tiltXPreMin];
        let yAlert=[r.tiltYMax,r.tiltYMin,r.tiltYPreMax,r.tiltYPreMin];
        let move1Alert=[r.move1Max,r.move1Min,r.move1PreMax,r.move1PreMin];
        let move2Alert=[r.move2Max,r.move2Min,r.move2PreMax,r.move2PreMin];
        let move3Alert=[r.move3Max,r.move3Min,r.move3PreMax,r.move3PreMin];
        let move4Alert=[r.move4Max,r.move4Min,r.move4PreMax,r.move4PreMin];
        switch(this.selChart){
          case 0:
            WarnValue=xAlert;
            break;
          case 1:
            WarnValue=yAlert;
            break;
          case 2:
            WarnValue=move1Alert;
            break;
          case 3:
             WarnValue=move2Alert;
            break;
          case 4:
             WarnValue=move3Alert;
            break;
          case 5:
            WarnValue=move4Alert;
            break;
            
        }
     
      if (this.selstartdate && this.selenddate) {
        if (this.coordinators && this.coordinators.length > 0) {
          this.$utils.showLoading();
          let i=0;
          api.getStationAvgDataByDate(this.station.id, this.coordinators[0].seqId, this.selstartdate, this.selenddate)
          .then(result => {
            this.$utils.hideLoading();
            if (result && result.length > 0) {
              result.forEach(r => {
                this.stationDatas.labels.push(r.date);
                displayData=[r.tilt_Avg_X,r.tilt_Avg_Y,r.move1_Avg,r.move2_Avg,r.move3_Avg,r.move4_Avg];
         //       console.info(displayData+"---"+this.selChart);
                this.stationDatas.datasets[0].data.push(this.toFixedFloat(displayData[this.selChart], 2));
                  if(i===0){minData=displayData[this.selChart];maxData=displayData[this.selChart];}
                  if(displayData[this.selChart]<minData){minData=displayData[this.selChart];}  
                  if(displayData[this.selChart]>maxData){maxData=displayData[this.selChart];}              
                  for(let j=0;j<4;j++){
                    this.stationDatas.datasets[j+1].data.push(WarnValue[j]); 
                   
                  }
                  if(minData>WarnValue[1]){
                    minData=WarnValue[1];
                  }
                  if(maxData<WarnValue[0]){
                    maxData=WarnValue[0];
                  }  
               i++;                
              });
            }
            this.drawChartData();
          }).catch(err => {
            this.$utils.hideLoading();
            this.$utils.toast(`获取数据失败: ${err.message}`);
          })
        } else {
          this.$utils.toast(`该油井还没有协调器`);
        }
      } else {
        this.$utils.toast(`请先选择开始日期及结束日期`);
      } 
       }).catch(err => {
        this.$utils.hideLoading();
        this.$utils.toast(`获取报警设置信息出错: ${err.message}`);
      })           
    },
    refreshDailyChartData(cStart,cEnd) {
      this.dailyDatas = null;
      
      this.dailyDatas = {
          labels: [],
          datasets: [{
              label: this.labelList[this.selChart], 
              backgroundColor: '#4DD0E1',
              borderColor: '#4DD0E1',
              pointBorderWidth: 4,
              data: [],
              fill: false
            }, {
            label: '报警', 
            backgroundColor: 'red',
            borderColor: 'red',
            borderDash: [10,3],
            radius:"0",
            borderWidth : 1,//线条的宽度
            data: [],
            fill: false
          }, {
            label:"",
            backgroundColor: 'red',
            borderColor: 'red',
            borderDash: [10,3],
            borderWidth : 1,//线条的宽度
            radius:"0",
            data: [],
            fill: false
          },{
            label: '预警', 
            backgroundColor: '#FF7F24',
            borderColor: '#FF7F24',
            borderDash: [10,3],
            borderWidth : 1,//线条的宽度
            radius:"0",
            data: [],
            fill: false
          },{
            label:"",
            backgroundColor: '#FF7F24',
            borderColor: '#FF7F24',
            borderDash: [10,3],
            borderWidth : 1,//线条的宽度
            radius:"0",
            data: [],
            fill: false
          }]          
          };
      this.$utils.showLoading();
      api.loadAlertSetting()
      .then(result => {
        this.$utils.hideLoading();
        this.asts = result;
        let r = this.asts[0];  
        let WarnValue=[0,0,0,0];
        let xAlert=[r.tiltXMax,r.tiltXMin,r.tiltXPreMax,r.tiltXPreMin];
        let yAlert=[r.tiltYMax,r.tiltYMin,r.tiltYPreMax,r.tiltYPreMin];
        let move1Alert=[r.move1Max,r.move1Min,r.move1PreMax,r.move1PreMin];
        let move2Alert=[r.move2Max,r.move2Min,r.move2PreMax,r.move2PreMin];
        let move3Alert=[r.move3Max,r.move3Min,r.move3PreMax,r.move3PreMin];
        let move4Alert=[r.move4Max,r.move4Min,r.move4PreMax,r.move4PreMin];
        switch(this.selChart){
          case 0:
            WarnValue=xAlert;
            break;
          case 1:
            WarnValue=yAlert;
            break;
          case 2:
            
            WarnValue=move1Alert;
            break;
          case 3:
            
            WarnValue=move2Alert;
            break;
          case 4:
            
            WarnValue=move3Alert;
            break;
          case 5:
          
            WarnValue=move4Alert;
            break;
        }
       
      if (this.seldate) {
        if (this.selcoor) {
          this.dailyData = [];
          this.$utils.showLoading();
          
          api.getStationDataByDay(this.selcoor.stationId, this.selcoor.seqId, cStart,cEnd)
          .then(result => {
            this.$utils.hideLoading();
            this.dailyData = result;
            

            if (this.dailyData && this.dailyData.length > 0) {
              for (let i = this.dailyData.length-1 ; i >= 0; i--) {
                let r = this.dailyData[i];            
             //   console.info(r);
                this.dailyDatas.labels.push(this.toTimeFormat(r.receivedAt));
                let displayData=[r.tilt1X,r.tilt1Y,r.move1,r.move2,r.move3,r.move4];
           //     console.info(displayData+"---"+this.selChart);
                this.dailyDatas.datasets[0].data.push(this.toFixedFloat(displayData[this.selChart], 2));
                  if(i===this.dailyData.length-1){minData=displayData[this.selChart];maxData=displayData[this.selChart];}
                  if(displayData[this.selChart]<minData){minData=displayData[this.selChart];}  
                  if(displayData[this.selChart]>maxData){maxData=displayData[this.selChart];} 
                  for(let j=0;j<4;j++){
                    this.dailyDatas.datasets[j+1].data.push(WarnValue[j]); 
                   
                  }
                  if(minData>WarnValue[1]){
                    minData=WarnValue[1];
                  }
                  if(maxData<WarnValue[0]){
                    maxData=WarnValue[0];
                  }  
                }
                    
              }
              this.drawDailyChart();
          }).catch(err => {
            this.$utils.hideLoading();
            this.$utils.toast(`获取历史数据出错: ${err.message}`);
          });
        } else {
          this.$utils.toast(`该油井还没有协调器`);
        }
      } else {
        this.$utils.toast(`请先选择日期`);
      }
       }).catch(err => {
        this.$utils.hideLoading();
        this.$utils.toast(`获取报警设置信息出错: ${err.message}`);
      })
    },

    toNumValue(v) {
      if (v && v != null && v != undefined) {
        return v.toFixed(2);
      }
      return '';
    },
    toFixedFloat(v, num) {
      return parseFloat(v.toFixed(num));
    },
    toTimeFormat(v) {
      let dt = moment(v);
      return dt.format('HH:mm:ss');
    },
    prevHourChart(){
      this.selHourNum=this.selHourNum-1;
      if(this.selHourNum<0) {
        this.$utils.toast(`已经是第一页`);
        this.selHourNum=0;
      }else{
        let startHour=this.selHourNum*2;
        let endHour=startHour+1;
        let prevCStart=this.seldate+" "+startHour+":00:00";
        let prevCEnd=this.seldate+" "+endHour+":59:59";
        this.refreshDailyChartData(prevCStart,prevCEnd);
      }
  
    },
    nextHourChart(){
      this.selHourNum=this.selHourNum+1;
      if(this.selHourNum>11) {
        this.$utils.toast(`已经是最后一页`);
        this.selHourNum=11;
      }else{
        let startHour=this.selHourNum*2;
        let endHour=startHour+1;
        let prevCStart=this.seldate+" "+startHour+":00:00";
        let prevCEnd=this.seldate+" "+endHour+":59:59";
        this.refreshDailyChartData(prevCStart,prevCEnd);
      }
    }
  },
  beforeMount: function() { 
    this.seldate = new Date().toISOString().substr(0, 10);  
    this.selstartdate= new Date(new Date().getTime() - 7*24*60*60*1000).toISOString().substr(0, 10);
    this.selenddate = new Date().toISOString().substr(0, 10);  
      // this.timer=setInterval(() => {
      //   this.nextHourChart();
      //   this.$utils.toast(this.selHourNum);
      // }, 1000); 
  },
  // Mount()  {
  //   if(this.timer){
  //     clearInterval(this.timer);
  //   }else{
  //     this.timer=setInterval(() => {
  //       this.nextHourChart();
  //     }, 15000);
  //   }

  // },
  activated: function() {    
    this.seltab = 'basic';
     
    this.loadCoordinators(); 
    
  },
  destroyed(){
    //    clearInterval(this.timer);
  },
};
</script>

<style scoped>
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
.item_value {
  align-items: flex-end;
  font-weight: 600;
}
.action {
  color: indigo;
  font-weight: 800;
}
.hot_icon {
  font-size: 14px;
}
.cus-icon {
  position: relative;
  z-index: 1;
}
.cus-icon::before {
  height: 17px;
  width: 17px;
  content: "";
  position: absolute;
  opacity: 0.54;
  z-index: -1;
  top: 0;
  left: 0;
  background-size: 17px 17px;
  background-repeat: no-repeat;
}
.upbtn {
  display: block;
  position: fixed; /* Fixed/sticky position */
  bottom: 70px; /* Place the button at the bottom of the page */
  right: 20px; /* Place the button 30px from the right */
  z-index: 99; /* Make sure it does not overlap */
  border: none; /* Remove1 borders */
}
</style>
