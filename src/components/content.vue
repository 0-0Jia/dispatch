<template>
  <div class="content-wrapper">
    <h1 style="text-align: center">两道批处理系统的两级调度-1</h1>
    <div class="content">
      <div class="title">作业队列</div>
      <table class="table table-hover">
        <tbody>
          <tr class="active">
            <td>作业名</td>
            <td>运行时间</td>
            <td>到达时间</td>
            <td>优先数</td>
          </tr>
          <tr v-for="(job, index) in inputWell" :key="index">
            <td>{{ job.name }}</td>
            <td>{{ job.runTimes }}</td>
            <td>{{ job.arrHours }}:{{ job.arrMins }}</td>
            <td>{{ job.priority }}</td>
          </tr>
        </tbody>
      </table>
      <div v-if="waiting[0] !== undefined">
      <div class="title">进程队列</div>
      <table class="table table-hover">
        <tbody>
          <tr class="active">
            <td>进程名</td>
            <td>开始时间</td>
            <td>实际运行时间</td>
            <td>优先数</td>
          </tr>
          <tr
            
            v-for="(item, index) in waiting"
            :key="index"
          >
            <td>{{ item.name }}</td>
            <td>{{ item.beginHours }}:{{ item.beginMins }}</td>
            <td>{{ item.runningHours }}:{{ item.runningMins }}</td>
            <td>{{ item.priority }}</td>
          </tr>
        </tbody>
      </table>
      </div>
      <div v-if="averageTime[0] !== undefined">
      <div class="title">完成队列</div>
      <table class="table table-hover">
        <tbody>
          <tr class="active">
            <td>作业名</td>
            <td>开始时间</td>
            <td>结束时间</td>
            <td>周转时间/min</td>
          </tr>
          <tr
            
            v-for="(item, index) in averageTime"
            :key="index"
          >
            <td>{{ item.name }}</td>
            <td>{{ item.beginHours }}:{{ item.beginMins }}</td>
            <td>{{ item.endHours }}:{{ item.endMins }}</td>
            <td>{{ item.turnAroundTime }}</td>
          </tr>
        </tbody>
      </table>
      </div>
      <div>平均周转时间/min：{{average}}</div>
      <div class="sub">
        <button class="btn btn-primary" @click="startProcess">运行</button>
      </div>
      <div class="container-fluid">
        <p>提交新作业</p>
        <div class="row">
          <label for="name" class="col-md-2 control-label">作业名</label>
          <input
            type="text"
            id="name"
            name=""
            class="col-md-3"
            v-model="newName"
          />
          <label for="runTimes" class="col-md-2 control-label">运行时间</label>
          <input
            type="number"
            id="runTimes"
            name=""
            class="col-md-3"
            v-model="newRunTimes"
          />
        </div>
        <div class="row">
          <label for="arrHours" class="col-md-2 control-label"
            >到达的小时</label
          >
          <input
            type="number"
            id="arrHours"
            name=""
            class="col-md-3"
            v-model="newArrHours"
          />
          <label for="arrMins" class="col-md-2 control-label">到达的分钟</label>
          <input
            type="number"
            id="arrMins"
            name=""
            class="col-md-3"
            v-model="newArrMins"
          />
        </div>
        <div class="row">
          <label for="priority" class="col-md-2 control-label">优先级</label>
          <input
            type="number"
            id="priority"
            name=""
            class="col-md-3"
            v-model="newpriority"
          />
        </div>
        <button class="btn btn-submit" @click="addNewJobIntoInputWell">
          submit
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      inputWell: [
        {
          name: "JOB1",
          arrHours: 10, // 到达时钟
          arrMins: "00", // 到达分钟
          runTimes: 40, // 需要的运行时间
          priority: 5, // 优先数
        },
        {
          name: "JOB2",
          arrHours: 10, // 到达时钟
          arrMins: 20, // 到达分钟
          runTimes: 30, // 需要的运行时间
          priority: 3,
        },
        {
          name: "JOB3",
          arrHours: 10, // 到达时钟
          arrMins: 30, // 到达分钟
          runTimes: 50, // 需要的运行时间
          priority: 4,
        },
        {
          name: "JOB4",
          arrHours: 10, // 到达时钟
          arrMins: 50, // 到达分钟
          runTimes: 20, // 需要的运行时间
          priority: 6,
        },
      ],

      waiting: [], // 就绪队列

      // 就绪队列有没变化，默认false状态不变
      change: false,

      // 用于记录就绪队列首元素被销毁时的结束时间
      record: [],
      recordBefore: { beforeH: 0, beforeM: 0, beforeruntime: 0 },
      key: true,

      // 完成队列数组
      averageTime: [],

      // 新作业
      newName: "",
      newRunTimes: 0,
      newArrHours: 0,
      newArrMins: 0,
      newpriority: 0,
    };
  },

  computed: {
    // 计算平均周转时间
    average: function () {
      let sum = this.averageTime.reduce(function (average, item) {
        return average + item.turnAroundTime;
      }, 0);
      let len = this.averageTime.length;
      let result = sum / len;
      return result;
    },
  },

  methods: {
    /**
     * 往输入井中增加新作业
     */
    addNewJobIntoInputWell: function () {
      // 新作业
      let newJob = {
        name: this.newName,
        arrHours: Number(this.newArrHours), // 到达时钟
        arrMins: Number(this.newArrMins), // 到达分钟
        runTimes: Number(this.newRunTimes), // 需要的运行时间
        priority: Number(this.newpriority),
      };
      this.inputWell.push(newJob);
      window.alert("提交成功");
    },
    /**
     * 开始执行
     */
    startProcess() {
      let _this = this;
      // 全局定时器
      let runTime = setInterval(startProcessing, 200);
      /**
       * 循环定时执行函数
       */
      function startProcessing() {

        // 如果作业队列和就绪队列都空就清除定时器
        if (
          _this.inputWell[0] === undefined &&
          _this.waiting[0] === undefined
        ) {
          _this.recordBefore.beforeM = 0;
          _this.recordBefore.beforeH = 0;
          _this.beforeruntime = 0;
          clearInterval(runTime);
        }

        // 就绪队列首进程运行结束前就绪队列中是否有新进程加入
        let processing;
        _this.addJobIntoWaiting();
        processing = _this.waiting[0];

        // 格式修改
        if(processing.arrMins == "00"){
          processing.arrMins == 0;
        }

        let h =
          _this.recordBefore.beforeH === 0
            ? processing.runningHours
            : _this.recordBefore.beforeH;
        let m =
          _this.recordBefore.beforeH === 0
            ? processing.runningMins
            : _this.recordBefore.beforeM;

        console.log("m:" + m);
        processing.trueM++;
        processing.runningHours = h + Math.floor((m + 1) / 60);
        processing.runningMins = (m + 1) % 60;

        console.log("运行进程的runningMins:" + processing.runningMins);

        let hours = processing.runningHours;
        let mins = processing.runningMins;

        console.log("运行进程的runningHours:" + hours);

        // // 进程结束标记
        // let over = (hours - processing.beginHours) * 60 + mins

        // 队首进程结束前
        console.log("进程当前的运行时间：" + processing.trueM);
        if (processing.trueM < processing.runTimes) {
          _this.recordBefore.beforeH = hours;
          _this.recordBefore.beforeM = mins;
          _this.recordBefore.beforeruntime = processing.runTimes;
          console.log(
            processing.name +
              "进程还在运行......." +
              hours +
              ":" +
              mins +
              " trueM:" +
              processing.trueM +
              " recordBefore.m" +
              _this.recordBefore.beforeM
          );
        } else if (processing.trueM >= processing.runTimes) {
          // 计算周转时间并且保存到averageTime
          let a = {
            name: processing.name,
            beginHours: processing.beginHours,
            beginMins:processing.beginMins,
            endHours: hours,
            endMins: mins,
            turnAroundTime:
              (hours - processing.beginHours) * 60 +
              (mins - processing.beginMins),
          };
          _this.averageTime.push(a);

          // 队首进程结束，释放资源

          processing.endHours = hours;
          processing.endMins = mins;
          _this.recordBefore.beforeH = hours;
          _this.recordBefore.beforeM = mins;
          _this.recordBefore.beforeruntime = processing.runTimes;
          let val = _this.waiting.splice(0, 1);
          _this.record.push(val);
        }
      }
    },
    // 把输入井的第一个作业添加进就绪队列，并为他分配内存外设
    addFirstJobToWaiting() {
      let _this = this;
      if (_this.inputWell[0] !== undefined) {
        // 输入井首元素
        let firstJob = _this.inputWell[0];
        console.log("addFirstJob：" + firstJob.arrMins);
        // 建立pcb
        let newPCB = _this.createPCB(firstJob, 0);
        // 添加进就绪队列
        _this.waiting.push(newPCB);
        // waiting改变就重排
        _this.waiting.sort(_this.minFirst("priority"));
      } else {
        console.log("作业队列已经空");
      }
    },
    // 建立pcb,会取得内存外设资源
    createPCB(item, index) {
      // 取得资源
      let _this = this;

      console.log(
        "waiting[0]:" + _this.waiting[0] + "item.arrMins:" + item.arrMins
      );
      // 进程的开始时间
      let beginH =
        _this.waiting[0] === undefined
          ? item.arrHours
          : _this.recordBefore.beforeH;
      let beginM =
        _this.waiting[0] === undefined
          ? item.arrMins
          : _this.recordBefore.beforeM;
      console.log("创建" + item.name + "的开始运行时间:" + beginM);
      // pcb
      let process = {
        name: item.name,
        runTimes: item.runTimes,
        priority: item.priority,
        beginHours: beginH, // 开始运行时
        beginMins: beginM, // 开始运行分
        runningHours: beginH, // 实时运行时间
        runningMins: beginM, // 实时运行时间
        endHours: 0,  // 结束时
        endMins: 0,   //结束分
        trueM: 0, 
      };

      console.log("process.runningMins:" + process.runningMins);

      // 删去输入井首元素
      _this.inputWell.splice(index, 1);
      return process;
    },

    /* 
    * 把作业添加进就绪队列
    */
    addJobIntoWaiting() {
      // 把输入井第一个作业添加进就绪队列，并为其分配内存外设
      let _this = this;
      if (_this.inputWell[0] !== undefined) {
        // 限制内存最多只能有两道作业
        if (_this.waiting.length < 2) {
          // 首进程
          if (_this.recordBefore.beforeH === 0) {
            console.log("添加首进程");
            _this.addFirstJobToWaiting(); // 添加首个进程
          } else {

            let first = 0;
            let index2 = 0;

            // 将最先到达以及运行时间最小的作业从输入井中添加进就绪队列
            _this.inputWell.forEach((item, index) => {
              if (
                item.arrHours < _this.recordBefore.beforeH ||
                (item.arrHours === _this.recordBefore.beforeH &&
                  item.arrMins <= _this.recordBefore.beforeM)
              ) {
                if (first === 0) {
                  index2 = index;
                  first = item;
                } else if (item.runTimes < first.runTimes) {
                  index2 = index;
                  first = item;
                }
              }
            });
            // 发生抢占现象
            if (first !== 0) {
              console.log(
                "取出输入井的元素名称：" + first.name + " 索引：" + index2
              );
              let newProcess = _this.createPCB(first, index2);
              console.log("添加抢占进程:" + newProcess.name);
              _this.waiting.push(newProcess);
              // waiting改变就重排
              _this.waiting.sort(_this.minFirst("priority"));
              console.log(_this.waiting[0].name + "是首进程");
            }
          }
        }
      }
    },
    minFirst(propertyName) {
      return function (obj1, obj2) {
        let val1 = obj1[propertyName];
        let val2 = obj2[propertyName];
        if (val1 < val2) {
          return -1;
        } else if (val1 > val2) {
          return 1;
        } else {
          return 0;
        }
      };
    },
  },
};
</script>

<style>
table {
  border: 1px solid rgba(179, 178, 178, 0.616);
}
.title {
  margin: 5px 0;
  font-size: 20px;
}
.content-wrapper {
  margin: 40px auto;
  z-index: -3;
  color: black;
}
.container-fluid {
  margin-top: 60px;
}
.content {
  width: 80%;
  margin: 0 auto;
}
.active {
  color: rgb(63, 2, 22);
}
.table-hover:hover {
  color: rgb(206, 115, 10);
}
input {
  color: dodgerblue;
}
.btn-submit {
  background-color: dodgerblue;
  color: white;
}
.sub {
  width: 100%;
  min-height: 30px;
  margin-bottom: 20px;
}
.row {
  margin: 20px 0;
}
</style>
