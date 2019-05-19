<template>
  <div>
    <new-meeting-form @added="addMeeting($event)"></new-meeting-form>
    <span v-if="meetings.length == 0">
               Brak zaplanowanych spotkań.
           </span>
    <h3 v-else>
      Zaplanowane zajęcia ({{ meetings.length }})
    </h3>
    <meetings-list :meetings="meetings"
                   :username="username"
                   @attend="addMeetingParticipant($event)"
                   @unattend="removeMeetingParticipant($event)"
                   @delete="deleteMeeting($event)"></meetings-list>
  </div>
</template>

<script>
    import NewMeetingForm from "./NewMeetingForm";
    import MeetingsList from "./MeetingsList";

    export default {
        components: {NewMeetingForm, MeetingsList},
        props: ['username'],
        data() {
            return {
                meetings: [],
                meetingslist: []
            };
        },
        created(){
        	this.fetchMeetings()
        },
        methods: {
        	fetchMeetings(){
        	this.$http.get('meetings')
                     .then(response => {
                         this.meetings = response.body;
                     })
                     .catch(response => {
                          console.error(response);
                     });
        	},
            addMeeting(meeting) {
                this.meetings.push(meeting);
                this.$http.post('meetings', meeting)
                this.fetchMeetings()
            },
            addMeetingParticipant(meeting) {
                meeting.participants.push(this.username);
                this.$http.post('meetings/'+ meeting.id +'/participants', {login:this.username})
                     .then(response => {
                          this.fetchMeetings()
                     })
                     .catch(response => {
                          console.log("Error :: couldn't add participant to meeting");
                     });
            },
            removeMeetingParticipant(meeting) {
                meeting.participants.splice(meeting.participants.indexOf(this.username), 1);
                this.$http.delete('meetings/'+ meeting.id + '/participants/' + this.username)
                     .then(response => {
                          this.fetchMeetings()
                     })
                     .catch(response => {
                          console.error("Error :: couldn't remove participant from the meeting");
                     });
            },
            deleteMeeting(meeting) {
            	this.meetings.splice(this.meetings.indexOf(meeting),1);
                console.log(meeting);
                this.$http.delete('meetings/'+ meeting.id)
                .then(response => {
                console.log(`${meeting} has been deleted!`);
                })
                .catch(response=>{
                console.error(response);
                });
            }
        }
    }
</script>