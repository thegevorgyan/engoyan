<template>
  <v-container fluid-grid-list-sm>
    <v-slide-y-transition mode="out-in">
      <v-layout row wrap>        
        <v-flex xs12 sm6>
          <v-menu
            ref="menu1"
            :close-on-content-click="false"
            v-model="menu1"
            :nudge-right="40"
            lazy
            transition="scale-transition"
            offset-y
            full-width
            max-width="290px"
            min-width="290px"
            >
            <v-text-field
              slot="activator"
              v-model="dateFormatted"
              label="Analysis Date"
              hint=""
              persistent-hint
              prepend-icon="event"
              @blur="date = parseDate(dateFormatted)"
            ></v-text-field>
            <v-date-picker v-model="date" no-title @input="menu1 = false"></v-date-picker>        
          </v-menu>
          <input type="text" v-model='percent'>
          <table>
            <tr v-for='items in dates'>
              <td><input type="text" v-model="items.day"></td>
              <td><input type="text" v-model="items.sum"></td>
            </tr>
          </table>
          <v-btn color="info" @click="calc">Calculate</v-btn>
        </v-flex>
        <v-flex xs12 sm6>
          <v-data-table
              :headers="headers"
              :items="dates"
              hide-actions
              class="elevation-1"
          >
          <template slot="items" slot-scope="props">
            <td >{{ props.item.day }}</td>
            <td v-if="props.item.sum<0" class="red--text">{{ props.item.sum }}</td>
            <td v-else class="green--text">{{ props.item.sum }}</td>
            <td v-if="props.item.sum<0" class="red--text">{{ props.item.pv }}</td>
            <td v-else class="green--text">{{ props.item.pv }}</td>
            <td class="text-xs-right">{{ props.item.fat }}</td>
          </template>
          </v-data-table>
          <span>Net Present Value:<strong>{{total_sum}}</strong></span>
        </v-flex>  
      </v-layout>
    </v-slide-y-transition>
  </v-container>
</template>

<script>
export default {
  data () {
    return {
        date: null,
        dateFormatted: null,
        menu1: false,
        menu2: false,
        percent:4,
        total_sum:0,
        headers: [
          {
            text: 'Date',
            align: 'left',
            value: 'name'
          },
          { text: 'Cash Flow', value: 'calories' },
          { text: 'PV', value: 'fat' }
        ],
        dates:[
          {
            day:'1/1/2019',
            sum:'-5000',
            pv: 0
          },
          {
            day:'1/1/2020',
            sum:'1000',
            pv: 0
          },
          {
            day:'1/1/2021',
            sum:'1000',
            pv: 0
          },
          {
            day:'1/1/2022',
            sum:'1000',
            pv: 0
          },
          {
            day:'1/1/2023',
            sum:'4000',
            pv: 0
          }
        ]
    }
  },
  computed: {
    computedDateFormatted () {
      return this.formatDate(this.date)
    }
  },
  watch: {
    date (val) {
      this.dateFormatted = this.formatDate(this.date)
    }
  },
  methods:{
    formatDate (date) {
      if (!date) return null
      const [year, month, day] = date.split('-')
      return `${month}/${day}/${year}`
    },
    parseDate (date) {
      if (!date) return null
      const [month, day, year] = date.split('/')
      return `${year}-${month.padStart(2, '0')}-${day.padStart(2, '0')}`    
    },
    calc : function(){
      var dateString = [],
          selectDate = [];

      for(let i=0; i<this.dates.length; i++){
         dateString.push(this.dates[i]);
         selectDate.push(new Date(dateString[i].day))
      }

      function Cons(value){
          var now = value;
          var start = new Date(now.getFullYear(), 0, 0);
          var diff = (now - start) + ((start.getTimezoneOffset() - now.getTimezoneOffset()) * 60 * 1000);
          var oneDay = 1000 * 60 * 60 * 24;
          var day = Math.floor(diff / oneDay);
          var arr = [now.getFullYear(), day];
          return arr;
      }

      var currentDate = new Cons(new Date(this.dateFormatted)),
          date1 = new Cons(selectDate[0]),
          date2 = new Cons(selectDate[1]),
          date3 = new Cons(selectDate[2]),
          date4 = new Cons(selectDate[3]),
          date5 = new Cons(selectDate[4]),
          ii = [
            (((date1[0] - currentDate[0])*365)-currentDate[1])+1,
            (((date2[0] - currentDate[0])*365)-currentDate[1])+1,
            (((date3[0] - currentDate[0])*365)-currentDate[1])+1,
            (((date4[0] - currentDate[0])*365)-currentDate[1])+1,
            (((date5[0] - currentDate[0])*365)-currentDate[1])+1,
          ];

      this.total_sum=0;

      for(let i=0; i<this.dates.length; i++){
        this.dates[i].pv = ((parseInt(this.dates[i].sum)) / (Math.pow(1 + (this.percent/100), ii[i]/365))).toFixed(2);
        this.total_sum += parseInt(this.dates[i].pv);
      }
    }
  }
}
</script>