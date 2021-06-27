<template>
<div>
  <el-container>
    <el-header>
      <h1>Administration</h1>
    </el-header>
    <el-main>
      <v-date-picker v-model='selectedDate'
      :attributes='attributes'
      /><br/>
      <el-button @click="selectedNum--; load()">&lt; Prev</el-button>
      <el-button @click="selectedNum++; load()">Next &gt;</el-button>
    </el-main>
    <el-main v-if="selectedDate">
      <h3>Number {{ story.number }}</h3>
      <el-form ref="form" :model="story" label-width="120px">

        <el-form-item label="Title">
          <el-input type="text" v-model="story.title"></el-input>
        </el-form-item>

        <el-form-item label="Prompt">
          <el-row>
            <el-col :span="12">
              <el-input type="textarea" autosize v-model="story.prompt"></el-input>
            </el-col>
            <el-col :span="12">
              <VueDocPreview
                :value="story.prompt"
                type="markdown"
              />
            </el-col>
          </el-row>

        </el-form-item>

        <el-form-item label="Content">
          <el-row>
            <el-col :span="12">
              <el-input type="textarea" autosize v-model="story.content"></el-input>
            </el-col>
            <el-col :span="12">
              <VueDocPreview
                  :value="story.content"
                  type="markdown"
              />
            </el-col>
          </el-row>
        </el-form-item>

        <el-form-item label="Tags">
          <el-row>
            <el-col>
              <el-input type="text" v-model="tags"></el-input>
            </el-col>
          </el-row>
        </el-form-item>

        <el-form-item label="Save">
          <el-button @click="save">Save</el-button>
        </el-form-item>

      </el-form>
    </el-main>
  </el-container>
</div>
</template>

<style>
.vm-markdown-html {
  line-height: normal;
}
</style>

<script>
import VueDocPreview from 'vue-doc-preview'

const startDate = new Date(2013,0,1, 0, 0, 0, 0);


function numberForDate(date) {
  const diff = date-startDate;
  let days = diff / 86400000;
  if (days > 365+31+28) days++;
  return (~~days)+1;
}

function dateForNumber(number) {
  let days = number-1;
  if (days > 365+31+28) days--;
  const date=new Date((+startDate)+(days*86400000));
  return date;
}

function dayOfYear(date) {
  const start = new Date(date.getFullYear(), 0, 0);
  const diff = date - start;
  const day = Math.floor(diff / 86400000);
  return day+1;
}

window.dateForNumber=dateForNumber;

const thisYear = (new Date).getFullYear();
export default {
  name: 'admin',
  components: {
    VueDocPreview,
  },
  data() {
    return {
      config: {
        apiURL: `${document.location.protocol}//${document.location.hostname}:3000`,
      },
      selectedDate: null,
      selectedNum: null,
      story: {
        year: thisYear,
        number: 0,
        day: 0,
        title: '',
        prompt: '',
        content: ''
      },
      tags: '',
      dates: [],
    }
  },
  computed: {
    years() {
      const list = [];
      const now = new Date();
      let year = 2013;
      while (year <= now.getFullYear()+1) {
        list.push(year);
        year++;
      }
      return list;
    },
    attributes() {
      const allDates = this.dates;

      const dates = [];
      let prev = null;
      let range = false;
      let cur = null;

      let n = 0;
      for (const date of allDates) {
        /* eslint-disable no-debugger */
        debugger
        if (n < 10) {
          console.log("prev", prev, "delta", date - prev);
          n++
        }
        if (prev && date - prev <= 86400000) {
          if (!range) {
            cur = { start: cur, end: date };
            range = true;
          } else {
            cur.end = date;
          }
        } else {
          if (cur) dates.push(cur);
          cur = date;
          range = false;
        }
        prev = date;
      }
      if (cur) dates.push(cur);

      console.log("using dates", dates)
      return [{
        key: 'stories',
        highlight: {
          start: { fillMode: 'solid', color: 'green' },
          base: { fillMode: 'solid', color: 'green' },
          end: { fillMode: 'solid', color: 'green' },
        },
        dates,
      }];
    }
  },
  methods: {
    init() {
      if (!this.config.loaded) return;
      this.updateCalendar();
      this.checkAdmin();
    },
    load() {
      const that = this;
      const number = this.selectedNum;
      fetch(`${this.config.apiURL}/story/${number}`, {
        credentials: "include"
      }).then(async (res)=> {
        try {
          that.story = await res.json();
        } catch (e) {
          that.story = {
            number,
            year: 0,
            day: 0,
            title: '',
            prompt: '',
            content: ''
          }
        }
      });
      fetch(`${this.config.apiURL}/tags/${number}`, {
        credentials: "include"
      }).then(async (res)=> {
        that.tags = (await res.json() ).join(', ');
      });
    },
    save() {
      const that = this;
      const date = dateForNumber(this.story.number);
      this.story.year = date.getFullYear();
      this.story.day = dayOfYear(date);
      fetch(`${this.config.apiURL}/story`, {
        method: 'POST',
        headers:  new Headers({
          'Content-Type': 'application/json; charset=utf-8'
        }),
        body: JSON.stringify(this.story),
        credentials: "include"
      }).then(async ()=> {
        that.updateCalendar();
        that.saveTags();
        this.$notify({
          title: 'Success',
          message: 'You did it!',
          type: 'success'
        });
      }).catch(()=>{
        this.$notify({
          title: 'Failure',
          message: 'Something fucked up...',
          type: 'error'
        });
      });

    },

    saveTags() {
      const that = this;
      fetch(`${this.config.apiURL}/tags/${this.story.number}`, {
        method: 'POST',
        headers:  new Headers({
          'Content-Type': 'application/json; charset=utf-8'
        }),
        body: JSON.stringify(this.tags.split(/,\s+/g)),
        credentials: "include",
      }).then(async ()=> {
        that.$notify({
          title: 'Success',
          message: 'Tags saved too!',
          type: 'success'
        });
      }).catch(()=>{
        that.$notify({
          title: 'Failure',
          message: 'Tags fail...',
          type: 'error'
        });
      });
    },

    updateCalendar() {
      const that = this;
      fetch(`${this.config.apiURL}/story`, {
        credentials: "include"
      }).then(async (res) => {
        const numbers = await res.json();
        const dates = [];
        for (const number of numbers) {
          dates.push(dateForNumber(number));
        }
        that.dates = dates;
      });
    },

    checkAdmin() {
      const that = this;
      fetch(`${this.config.apiURL}/account`, {
        credentials: "include"
      }).then(async (res) => {
        try {
          const account = await res.json();
          if (!account || account.email !== 'admin@internutter.org') {
            document.location = `${this.config.apiURL}/sign-in`;
          }
        } catch (e) {
          that.$notify({
            title: 'Failure',
            message: 'Admin fail...',
            type: 'error'
          });
        }
      });
    }
  },
  watch: {
    selectedDate() {
      this.selectedNum =  numberForDate(this.selectedDate);
      this.load();
    }
  },
  created() {
    fetch('config.json').then(async(res) => {
      try {
        this.config = await res.json();
        this.config.success = true;
      } catch (e) {
	this.config.success = false;
      } finally {
        this.config.loaded = true;
      }
      this.init();
    });
  },
  mounted() {
    this.init();
  },
}
</script>
