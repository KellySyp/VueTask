<template>
  <div class="tasks">
	<div class="container">
		<form @submit.prevent="addTask">
			<input type="text" placeholder="Enter a new task..." v-model="task" id="newTask">
		</form>
		<div id="sorting">
			<i>Search by...</i>
			<input type="text" placeholder="Keyword" v-model="keyword" v-on:keyup="searchKeyword" id="Keyword">
			Date Range: 
			<input type="date" v-model="dateStart" v-on:change="searchDate">
			<input type="date" v-model="dateEnd" v-on:change="searchDate">
			View All:
			<label class="switch">
				<input type="checkbox" v-model="setShowAll" v-on:change="showAll">
				<span class="slider round"></span>
			</label>
			<div id="sortStatus">{{sortMessage}}</div>
		</div>
	</div>
	<div class="holder">
		<ul>
			<transition-group name="list" enter-active-class="animated bounceInUp" leave-active-class="animated bounceOutDown">
				<li v-for="(data, index) in tasks" :key='index' v-if="data.show" 
					v-bind:style= "{'background-color': getColor(data)}">
				<!-- Task Name -->
					<div v-bind:class="{complete: data.complete}">{{data.task}}</div>
				<!-- Task Date -->
					<i class="subtitle">{{data.dateAdd.toLocaleDateString("en-US")}}</i>
				<!-- Task Functions -->
					<i class="fa fa-trash" v-on:click="delItem(index)"></i>
					<i class="fa fa-check" v-on:click="markComplete(index)"></i>
				<!-- Task Description -->
					<div class="desc">
						<textarea v-model="data.description" v-on:change="persist">{{data.description}}</textarea>
					</div>
				</li>
			</transition-group>
		</ul>
	</div>
  </div>
</template>

<script>
	export default {
		name: 'Tasks',
		data(){
			return{
				setShowAll: false,
				task: '',  // Add this line
				keyword: '',  // Add this line
				dateStart: '',
				dateEnd: '',
				sortMessage: 'Showing outstanding tasks',
				tasks: []
			}
		},
		mounted(){
			if(localStorage.tasks){
				this.tasks = JSON.parse(localStorage.tasks);
				//Sets date to JS format
				this.tasks.forEach(function(task){
					var dateStr = task.dateAdd;
					var Tcomplete = task.complete;
					task.dateAdd = new Date(dateStr);
					if(Tcomplete)task.show = false;
				});
			}
		},
		watch: {
			tasks(newTasks){
				localStorage.setItem("tasks", JSON.stringify(this.tasks));
			}
		},
		methods:{
			persist(){
				localStorage.setItem("tasks", JSON.stringify(this.tasks));
			},
			addTask(){
				this.tasks.push({task: this.task,
								description: "",
								dateAdd: new Date(),
								complete: false,
								show: true});
				this.task = '';
			},
			markComplete(id) {
				this.tasks[id].complete = !this.tasks[id].complete;
				this.persist();
			},
			delItem(id){
				if(confirm("Are you sure you want to delete?")){
					this.tasks.splice(id,1);
				}
			},
			showAll(){
				var TsetShowAll = this.setShowAll;
				if(TsetShowAll){
					this.sortMessage = 'Showing all tasks';
				}else{
					this.sortMessage = 'Showing outstanding tasks';
				}
				this.tasks.forEach(function(task){
					if(TsetShowAll){
						task.show = true;
					}else{
						if(!task.complete){
							task.show = true;
						}else{
							task.show = false;
						}
					}
				});
			},
			searchKeyword(){
				var Tkeyword = this.keyword.toLowerCase();
				this.sortMessage = 'Showing tasks containing '+this.keyword;
				this.tasks.forEach(function(task){
					var tname = task.task.toLowerCase();
					var tdesc = task.description.toLowerCase();
					if(tname.indexOf(Tkeyword) >= 0 || tdesc.indexOf(Tkeyword) >= 0 ){
						task.show = true;
					}else{
						task.show = false;
					}
				});
				if(Tkeyword.length < 1)this.showAll();
			},
			searchDate(){
				this.sortMessage = 'Showing tasks between '+this.dateStart+' and '+this.dateEnd;
				var TdateStart = new Date(this.dateStart);
				var TdateEnd = new Date(this.dateEnd);
				//This part is necessary to include the entire end date. Technically the date range ends at midnight the next day.
				TdateEnd.setDate(TdateEnd.getDate() + 1);
				this.tasks.forEach(function(task){
					var tdate = task.dateAdd;
					if(tdate >= TdateStart && tdate <= TdateEnd ){
						task.show = true;
					}else{
						task.show = false;
					}
				});
			},
			//Changes background color of tasks based on complete status or days added
			getColor(task){
				var today = new Date();
				var daysBtwn = (today - task.dateAdd)/24/60/60/1000;
				var color = '#E0EDF4';
				if(task.complete){
					color = '#e3f4e0'
				//If the task is older than 10 days, the task starts turning red. The longer it's out there, the darker it gets.
				}else if(daysBtwn >= 10){
					var oc = 255-(daysBtwn-10);
					color = 'rgb(255,'+oc+','+oc+')';
				}
				return color;
			}
		}
	}
</script>

<style scoped>
@import "https://maxcdn.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css"; 
@import "https://cdn.jsdelivr.net/npm/animate.css@3.5.1";
input {
	border: 0;
	padding: 20px;
}
#newTask, #Keyword{
	width: calc(100% - 40px);
}
#newTask{
	padding: 20px;
	margin-top: 20px;
	font-size: 1.3em;
}

#sorting{
	padding: 10px;
	margin-top: 20px;
	text-align: center;
}
#sorting input{
	margin: 5px;
}

#sortStatus{
	padding: 20px;
	background-color: #fcffcc;
	margin-top: 20px;
}

textarea{
	width: 100%;
	margin-top: 10px;
	height: 6em;
	resize: none;
	rows: 4;
	padding: 5px;
}
.holder {
	background: #fff;
}
ul {
	margin: 0;
	padding: 0;
	list-style-type: none;
}
ul li {
	padding: 20px;
	font-size: 1.3em;
	margin-bottom: 2px;
	color: #3E5252;
}
.container {
	box-shadow: 0px 0px 40px lightgray;
}
.fa{
	float: right;
	cursor: pointer;
	padding-left: 10px;
}
.complete{
	text-decoration: line-through;
}
.subtitle{
	font-size: medium;
}

/* ***************** */
/* Toggle switch CSS */
/* ***************** */

.switch {
	margin-left: 20px;
	position: relative;
	display: inline-block;
	width: 60px;
	height: 34px;
}

.switch input { 
	opacity: 0;
	width: 0;
	height: 0;
}

.slider {
	position: absolute;
	cursor: pointer;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	background-color: #ccc;
	-webkit-transition: .4s;
	transition: .4s;
}

.slider:before {
	position: absolute;
	content: "";
	height: 26px;
	width: 26px;
	left: 4px;
	bottom: 4px;
	background-color: white;
	-webkit-transition: .4s;
	transition: .4s;
}

input:checked + .slider {
	background-color: #2196F3;
}

input:focus + .slider {
	box-shadow: 0 0 1px #2196F3;
}

input:checked + .slider:before {
	-webkit-transform: translateX(26px);
	-ms-transform: translateX(26px);
	transform: translateX(26px);
}

/* Rounded sliders */
.slider.round {
	border-radius: 34px;
}

.slider.round:before {
	border-radius: 50%;
}
</style>
