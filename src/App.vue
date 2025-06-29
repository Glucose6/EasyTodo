<template>
  <div id="app">
    <h1>EasyTodo</h1>
    <div id="newTask">
        <input type="text" v-model="newTaskName" placeholder="Add a new task" @keyup.enter="addTask">
        <button @click="addTask">+ Add Task</button>
    </div>

    <div class="columns">
      
      <div class="column">
        <h2>Todo</h2>
        <ul>
          <li v-for="(task, index) in todoTasks" :key="index" class="task-item">
            <span>{{ task.name }}</span>
            <div>
              <button style="background-color: #17abe3;" @click="editTask(task)"><img src="/edit.png"></button>
              <button style="background-color: #e9270e;" @click="showRemoveConfirmation(task)"><img src="/delete.png"></button>
              <button disabled><img src="/arrow-left-s-line.png"></button>
              <button @click="moveTask(task, todoTasks, doingTasks)"><img src="/arrow-right-s-line.png"></button>
            </div>
          </li>
        </ul>
      </div>

      <div class="column">
        <h2>Doing</h2>
        <ul>
          <li v-for="(task, index) in doingTasks" :key="index" class="task-item">
            <span>{{ task.name }}</span>
            <div>
              <button style="background-color: #17abe3;" @click="editTask(task)"><img src="/edit.png"></button>
              <button style="background-color: #e9270e;" @click="showRemoveConfirmation(task)"><img src="/delete.png"></button>
              <button @click="moveTask(task, doingTasks, todoTasks)"><img src="/arrow-left-s-line.png"></button>
              <button @click="moveTask(task, doingTasks, doneTasks)"><img src="/arrow-right-s-line.png"></button>
            </div>
          </li>
        </ul>
      </div>

      <div class="column">
        <h2>Done</h2>
        <ul>
          <li v-for="(task, index) in doneTasks" :key="index" class="task-item">
            <span>{{ task.name }}</span>
            <div>
              <button style="background-color: #17abe3;" @click="editTask(task)"><img src="/edit.png"></button>
              <button style="background-color: #e9270e;" @click="showRemoveConfirmation(task)"><img src="/delete.png"></button>
              <button @click="moveTask(task, doneTasks, doingTasks)"><img src="/arrow-left-s-line.png"></button>
              <button disabled><img src="/arrow-right-s-line.png"></button>
            </div>
          </li>
        </ul>
      </div>
      
    </div>

    <!-- 添加: 编辑任务对话框 -->
    <div v-if="editingTask" class="edit-dialog">
      <div class="dialog-content">
        <h2>Edit Task</h2>
        <div class="edit-form">
          <div>
            <span>taskname</span>
          </div>
          <div>
            <input type="text" v-model="editingTaskCopy.name" placeholder="Edit task name">
          </div>
        </div>
        <div class="edit-form">
          <div>
            <span>description</span>
          </div>
          <div>
            <textarea v-model="editingTaskCopy.content" placeholder="description of task"></textarea>
          </div>
        </div>
        <!-- 添加: 子任务输入框 -->
        <h3>Subtasks</h3>
        <div class="edit-form">
          <div>
            <input type="text" v-model="newSubtaskName" placeholder="Add a subtask" @keyup.enter="addSubtask">
          </div>
          <div>
            <button @click="addSubtask" id="addSubtask">Add</button>
          </div>
        </div>
        <!-- 添加: 子任务显示区域 -->
        <ul class="subtasklists">
          <li v-for="(subtask, index) in editingTaskSubtasks" :key="index">
            <div class="subtasks">
              <input type="checkbox" v-model="subtask.completed" />
              <span :style="{ textDecoration: subtask.completed ? 'line-through' : 'none' }">{{ subtask.name }}</span>
            </div>
            <button @click="removeSubtask(index)">Remove</button>
          </li>
        </ul>
        <button @click="saveTask">Save</button>
        <button @click="cancelEdit">Cancel</button>
      </div>
    </div>

    <!-- 添加: 确认删除对话框 -->
    <div v-if="showConfirmDialog" class="confirm-dialog">
      <div class="dialog-content">
        <h2>Confirm Deletion</h2>
        <p>Are you sure you want to delete this task?</p>
        <div>
          <button @click="confirmRemoveTask">Yes</button>
          <button @click="cancelRemoveTask">No</button>
        </div>
      </div>
    </div>

    <!-- 添加: 导入导出按钮 -->
    <div id="importExportButtons">
        <button @click="exportTasks">Export Tasks</button>
        <input type="file" @change="importTasks" accept=".json" style="display: none;" ref="fileInput">
        <button @click="triggerFileInput">Import Tasks</button>
    </div>

  </div>
</template>

<script>
export default {
  data() {
    return {
      newTaskName: '',
      todoTasks: [],
      doingTasks: [],
      doneTasks: [],
      editingTask: null, // 用于存储当前正在编辑的任务
      editingTaskCopy: null, // 用于存储当前正在编辑的任务的副本
      editingTaskSubtasks: [], // 用于存储当前正在编辑的任务的子任务，改为对象数组
      newSubtaskName: '', // 用于存储新的子任务名称
      showConfirmDialog: false, // 添加: 确认对话框显示状态
      taskToRemove: null // 添加: 需要删除的任务
    };
  },
  methods: {
    addTask() {
      if (this.newTaskName.trim()) {
        // 检查是否存在相同任务名的任务
        const taskExists = this.todoTasks.some(task => task.name === this.newTaskName) ||
                           this.doingTasks.some(task => task.name === this.newTaskName) ||
                           this.doneTasks.some(task => task.name === this.newTaskName);
        if (taskExists) {
          alert('任务已经存在'); // 提示用户任务已经存在
          return; // 不添加任务
        }
        this.todoTasks.push({ name: this.newTaskName, content: '', subtasks: [] }); // 初始化子任务为对象数组
        this.newTaskName = '';
      }
    },
    moveTask(task, fromList, toList) {
      const taskIndex = fromList.indexOf(task);
      if (taskIndex !== -1) {
        fromList.splice(taskIndex, 1);
        toList.push(task);
      }
    },
    showRemoveConfirmation(task) { // 添加: 显示确认对话框
      this.taskToRemove = task;
      this.showConfirmDialog = true;
    },
    confirmRemoveTask() { // 添加: 确认删除任务
      if (this.taskToRemove) {
        let taskList = null;
        if (this.todoTasks.includes(this.taskToRemove)) {
          taskList = this.todoTasks;
        } else if (this.doingTasks.includes(this.taskToRemove)) {
          taskList = this.doingTasks;
        } else if (this.doneTasks.includes(this.taskToRemove)) {
          taskList = this.doneTasks;
        }
        if (taskList) {
          const taskIndex = taskList.indexOf(this.taskToRemove);
          if (taskIndex !== -1) {
            taskList.splice(taskIndex, 1);
          }
        }
        this.cancelRemoveTask(); // 关闭确认对话框
      }
    },
    cancelRemoveTask() { // 添加: 取消删除任务
      this.showConfirmDialog = false;
      this.taskToRemove = null;
    },
    editTask(task) {
      this.editingTask = task;
      this.editingTaskCopy = { ...task, content: task.content || '' }; // 确保 content 属性存在
      this.editingTaskSubtasks = task.subtasks ? [...task.subtasks] : []; // 初始化子任务
    },
    saveTask() {
      if (this.editingTask) {
        if (this.editingTaskCopy.name.trim()) {
          this.editingTask.name = this.editingTaskCopy.name; // 将副本的修改应用到原始任务对象上
          this.editingTask.content = this.editingTaskCopy.content; // 更新 content 属性
          this.editingTask.subtasks = this.editingTaskSubtasks; // 更新子任务
        } else {
          alert('任务名称不能为空'); // 提示用户任务名称不能为空
          return; // 不更新任务名称
        }
      }
      this.editingTask = null;
      this.editingTaskCopy = null;
      this.editingTaskSubtasks = []; // 清空子任务
    },
    cancelEdit() { 
      this.editingTask = null; // 关闭对话框
      this.editingTaskCopy = null; // 清除副本
    },
    findTask(task) { // 添加: 查找任务在哪个列表中
      return this.todoTasks.find(t => t === task) ||
             this.doingTasks.find(t => t === task) ||
             this.doneTasks.find(t => t === task);
    },
    addSubtask() {
      if (this.newSubtaskName.trim()) {
        // 检查是否存在相同子任务名的子任务
        const subtaskExists = this.editingTaskSubtasks.some(subtask => subtask.name === this.newSubtaskName);
        if (subtaskExists) {
          alert('子任务已经存在'); // 提示用户子任务已经存在
          return; // 不添加子任务
        }
        this.editingTaskSubtasks.push({ name: this.newSubtaskName, completed: false }); // 添加子任务时设置完成状态为false
        this.newSubtaskName = '';
      }
    },
    removeSubtask(index) {
      this.editingTaskSubtasks.splice(index, 1);
    },
    // 添加: 导出任务功能
    exportTasks() {
      const tasks = {
        todoTasks: this.todoTasks,
        doingTasks: this.doingTasks,
        doneTasks: this.doneTasks
      };
      const jsonString = JSON.stringify(tasks, null, 2);
      const blob = new Blob([jsonString], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'tasks.json';
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      URL.revokeObjectURL(url);
    },
    // 添加: 触发文件输入框
    triggerFileInput() {
      this.$refs.fileInput.click();
    },
    // 添加: 导入任务功能
    importTasks(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          try {
            const tasks = JSON.parse(e.target.result);
            this.todoTasks = tasks.todoTasks || [];
            this.doingTasks = tasks.doingTasks || [];
            this.doneTasks = tasks.doneTasks || [];
          } catch (error) {
            alert('Failed to import tasks. Please ensure the file is a valid JSON.');
          }
        };
        reader.readAsText(file);
      }
    }
  }
};
</script>

<style scoped>
*{
  text-decoration: none;
  list-style: none;
  margin: 0;
  padding: 0;
}

.columns {
  display: flex;
  justify-content: space-evenly;
  background-color: #f7f7f7;
  width: 96vw;
}

.column {
  width: 30%;
  padding: 10px 0;
  background-color: #fff;
  border-top-left-radius: 8px;
  border-top-right-radius: 8px;
}

.column > h2 {
  background-color: #000;
  color: #fff;
  padding: 10px;
  border-top-left-radius: 8px;
  border-top-right-radius: 8px;
}

.column > ul {
  height: 50vh;
  max-width: 100%;
  background-color: #fff;
  border-bottom-right-radius: 8px;
  border-bottom-left-radius: 8px;
}

.task-item {
  display: flex; 
  justify-content: space-between; 
  align-items: center;
  height: 4vh;
  padding: 1vh;
  border-bottom: 1px solid #ccc;
}

.task-item > span {
  width: 50%;
  text-align: start;
}

.task-item > div {
  display: flex; 
  width: 50%;
  justify-content: space-between;
}

.task-item > div > button {
  width: 16%;
  height: 4vh;
  background-color: #000;
  border: #000 solid 0;
  border-radius: 8px;
  color: #fff;
  cursor: pointer;
}

.task-item > div > button:disabled{
  cursor: not-allowed;
}

.task-item > div img{
  width: 100%;
  height: 100%;
}

#app {
  height: 100vh;
  width: 96vw;
  margin: 0 auto; 
  padding: 2vh;
  text-align: center;
  background-color: #f7f7f7;
  display: flex;
  flex-direction: column;
}

#newTask {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 96vw;
}

#newTask > input {
  width: 60vw;
  height: 4vh;
  font-size: 20px;
  padding: 2vh;
  margin: 10px 0;
}

#newTask > button {
  width: 20vw;
  height: 8.75vh;
  font-size: 20px;
  margin: 10px 0;
  box-sizing: border-box;
  background-color: #000;
  color: #fff;
}

.edit-dialog {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.dialog-content {
  background-color: #fff;
  padding: 20px;
  width: 32vw;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.dialog-content h2 {
  margin-bottom: 10px;
}
.dialog-content h3 {
  margin-bottom: 10px;
}

.edit-form {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.edit-form>span {
  width: 8vw;
}

.dialog-content input {
  width: 24vw;
  padding: 1em 1em;
  margin-bottom: 10px;
  border: 1px solid #444;
  border-radius: 4px;
  font-size: 1em;
  background-color: #1e1e1e;
  color: white;
  cursor: text;
  height: 100%;
}

.dialog-content textarea {
  width: 24vw;
  padding: 1em 1em;
  margin-bottom: 10px;
  border: 1px solid #444;
  border-radius: 4px;
  font-size: 1em;
  background-color: #1e1e1e;
  color: white;
  cursor: text;
  height: 100%;
  resize: none; /* 禁止通过拖动改变大小 */
  white-space: pre-wrap; /* 自动换行 */
  overflow-wrap: break-word; /* 确保文字溢出时自动换行 */
}

.dialog-content button {
  border-radius: 8px;
  border: 1px solid transparent;
  padding: 0.6em 1.2em;
  font-size: 1em;
  font-weight: 500;
  font-family: inherit;
  background-color: #000;
  color: white;
  cursor: pointer;
  transition: background-color 0.25s;
  height: 38px;
  margin-right: 10px;
}

#addSubtask{
  height: 100%;
  padding: 1em;
  margin-top: -10px;
}

.dialog-content button:hover {
  background-color: #eee;
  color:#000;
}

.subtasklists {
  display: flex;
  flex-direction: column; /* 确保子任务列表垂直排列 */
  padding: 1em;
}

.subtasklists li {
  display: flex; /* 使用 flex 布局 */
  align-items: center; /* 垂直居中对齐 */
  margin-bottom: 5px; /* 添加一些间距 */
  padding: 2px;
  justify-content: space-between;
  width: 100%;
  border-radius: 8px;
}

.subtasks {
  display: flex;
  width: 100%; /* 调整宽度为100% */
  justify-content: space-between;
}

.subtasklists li input[type="checkbox"] {
  width: 20px;
  height: 20px;
  cursor: pointer;
}

.subtasklists li span{
  width: 20vw;
  height: 100%;
  text-align: start;
}

.subtasklists li input[type="checkbox"]:checked{
  color: #000;
}

.subtasklists li input[type="checkbox"]:checked + span {
  text-decoration: line-through; /* 勾选时文本被划掉 */
}

.confirm-dialog {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.confirm-dialog .dialog-content {
  background-color: #fff;
  padding: 20px;
  width: 30vw;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.confirm-dialog .dialog-content h2 {
  margin-bottom: 10px;
}

.confirm-dialog .dialog-content p {
  margin-bottom: 20px;
}

.confirm-dialog .dialog-content button {
  margin-right: 10px;
}

#importExportButtons {
  margin-top: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 96vw;
}

#importExportButtons > button {
  width: 20vw;
  height: 8.75vh;
  font-size: 20px;
  box-sizing: border-box;
  background-color: #000;
  color: #fff;
}
</style>