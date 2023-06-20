<template>
  <div>
    <NewMeetingForm @added="addNewMeeting"></NewMeetingForm>
    <span v-if="meetings.length === 0">Brak zaplanowanych spotkań.</span>
    <h3 v-else>Zaplanowane zajęcia ({{ meetings.length }})</h3>
    <MeetingsList :meetings="meetings"
                  :username="username"
                  @attend="addMeetingParticipant"
                  @unattend="removeMeetingParticipant"
                  @delete="deleteMeeting"></MeetingsList>
  </div>
</template>

<script>
import NewMeetingForm from "./NewMeetingForm";
import MeetingsList from "./MeetingsList";
import axios from "axios";

export default {
  components: { NewMeetingForm, MeetingsList },
  props: { username: String },
  data() {
    return {
      meetings: []
    };
  },
  created() {
    this.fetchMeetings();
  },
  methods: {
    fetchMeetings() {
      axios.get("/api/meetings")
        .then(response => {
          this.meetings = response.data;
        })
        .catch(error => {
          console.error(error);
        });
    },
    addNewMeeting(meeting) {
      axios.post("/api/meetings", meeting)
        .then(response => {
          this.meetings.push(response.data);
        })
        .catch(error => {
          console.error(error);
        });
    },
    addMeetingParticipant(meeting) {
      axios.put(`/api/meetings/${meeting.id}/participants`, { username: this.username })
        .then(response => {
          meeting.participants.push(response.data.username);
        })
        .catch(error => {
          console.error(error);
        });
    },
    removeMeetingParticipant(meeting) {
      axios.delete(`/api/meetings/${meeting.id}/participants/${this.username}`)
        .then(response => {
          const index = meeting.participants.indexOf(this.username);
          if (index !== -1) {
            meeting.participants.splice(index, 1);
          }
        })
        .catch(error => {
          console.error(error);
        });
    },
    deleteMeeting(meeting) {
      axios.delete(`/api/meetings/${meeting.id}`)
        .then(response => {
          const index = this.meetings.indexOf(meeting);
          if (index !== -1) {
            this.meetings.splice(index, 1);
          }
        })
        .catch(error => {
          console.error(error);
        });
    }
  }
};
</script>