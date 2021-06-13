<template>
<div>
  <el-container>
    <el-header>
      <h1>Administration</h1>
    </el-header>
    <el-main>
      <v-date-picker v-model='selectedDate'
      :attributes='attributes'
      />
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
              <el-input type="textarea" v-model="story.prompt"></el-input>
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

const apiURL = 'http://localhost:3000';
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
      selectedDate: null,
      story: {
        year: thisYear,
        number: 0,
        day: 0,
        title: '',
        prompt: '',
        content: ''
      },
      tags: '',
      attributes: [],
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
    }
  },
  methods: {
    load() {
      const that = this;
      const number = numberForDate(this.selectedDate);
      fetch(`${apiURL}/story/${number}`).then(async (res)=> {
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
      fetch(`${apiURL}/tags/${number}`).then(async (res)=> {
        that.tags = (await res.json() ).join(', ');
      });
    },
    save() {
      const that = this;
      const date = dateForNumber(this.story.number);
      this.story.year = date.getFullYear();
      this.story.day = dayOfYear(date);
      fetch(`${apiURL}/story`, {
        method: 'POST',
        headers:  new Headers({
          'Content-Type': 'application/json; charset=utf-8'
        }),
        body: JSON.stringify(this.story),
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
      fetch(`${apiURL}/tags/${this.story.number}`, {
        method: 'POST',
        headers:  new Headers({
          'Content-Type': 'application/json; charset=utf-8'
        }),
        body: JSON.stringify(this.tags.split(/,\s+/g)),
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
      fetch(`${apiURL}/story`).then(async (res)=> {
        const list = [];
        const numbers = await res.json();
        const dates = [];
        for (const number of numbers) {
          dates.push(dateForNumber(number));
        }
        list.push({
          key: 'stories',
          highlight: 'green',
          dates,
        });
        that.attributes = list;
      });
    }
  },
  watch: {
    selectedDate() {
      this.load();
    }
  },
  mounted() {
    this.updateCalendar();
  }
}
</script>