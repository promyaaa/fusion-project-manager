<template>
    <div class="container">
        <project-nav v-on:get-project="setProject"></project-nav>
        <div class="lists calendar-container border-for-nav">
            <div class="row">
                <div class="col-12">
                	<FullCalendar
                	   	ref="fullCalendar"
					  	:plugins="calendarPlugins"
					  	:weekends="false"
					  	:eventLimit="true"
					  	:events="events"
					  	:header="header"
					  	@datesRender="handleMonthChange"
					  	@eventClick="showDetails"
					/>
                </div>
            </div>
        </div>
    </div>
</template>

<script>

import FullCalendar from '@fullcalendar/vue';
import dayGridPlugin from '@fullcalendar/daygrid';
// import timeGridPlugin from '@fullcalendar/timegrid';
import ProjectNav from './partials/ProjectNavComponent.vue';



export default {
	components: {
		FullCalendar, // make the <FullCalendar> tag available
		ProjectNav,
	},
	data() {
		return {
		    calendarPlugins: [ dayGridPlugin ], // timeGridPlugin
		    events: [],
			todos: [],
			header: {
		        left: 'prev,next today',
		        center: 'title',
		        right: 'dayGridMonth' //,timeGridWeek,timeGridDay' //,listMonth'
		    },
		};
	},
	methods: {
		showDetails(event) {
			// console.log(event);
			// alert(event.event.title);
		},
		handleMonthChange(arg) {
			var startDate = arg.view.activeStart;
			var endDate = arg.view.activeEnd;
			
			if(arg.view.type === 'dayGridMonth') {
				this.fetchProjectTodos(startDate, endDate);
			}
		},
		setProject( project ) {
		    this.project = project;
		},
		fetchProjectTodos(s, e) {
			var date = new Date();

		    var vm = this,
			    data = {
			        action: 'fpm-get-todos-for-calendar',
			        project_id: vm.$route.params.projectid,
			       	start_date: s.getFullYear() + '-' + s.getMonth() + '-' + s.getDate(),
			        end_date: e.getFullYear() + '-' + (e.getMonth()+1) + '-' + e.getDate(),
			        nonce: fpm.nonce,
			    };


		    jQuery.post( fpm.ajaxurl, data, function( resp ) {
		        if ( resp.success ) {
		            vm.todos = resp.data;
		            for (var i=0; i < vm.todos.length; i++) {
		            	if (!_.find(vm.events, {'id': vm.todos[i].ID})) {
		            		vm.events.push({
			            		id: vm.todos[i].ID,
			            		title: vm.todos[i].todo,
			            		start: vm.todos[i].created,
			            		end: vm.todos[i].due_date || '',
			            	});
		            	}
		            }
		        }
		    });
		}
	},
}

</script>

<style>

@import '~@fullcalendar/core/main.css';
@import '~@fullcalendar/daygrid/main.css';
@import '~@fullcalendar/timegrid/main.css';

.lists.calendar-container {
	overflow: unset;
}

</style>