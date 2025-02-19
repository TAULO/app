<template>
    <div class="container d-flex flex-column align-items-center">
        <!-- <div class="display-6 mb-5 text-center">Kommende begivenheder</div> -->
        <!-- ADMIN -->
        <button v-if="user.loggedIn" class="btn btn-warning m-3" data-bs-toggle="modal"
            data-bs-target="#addEventModal" @click="openAddEventModal()">Tilføj ny begivenhed
        </button>
        <!-- modal -->
        <div class="modal modal-lg fade" id="addEventModal">
            <div class="modal-dialog modal-dialog-scrollable">
                <div class="modal-content">
                    <div class="modal-header">
                        <h1 v-if="!this.shouldUpdate" class="modal-title fs-5" id="addEventModal">Tilføj ny begivenhed</h1>
                        <h1 v-else class="modal-title fs-5" id="addEventModal">Opdater begivenhed</h1>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                    <div class="modal-body">
                        <div class="input-group mb-3">
                            <span class="input-group-text" id="basic-addon1" style="min-width: 135px;">Overskrift</span>
                            <input type="text" class="form-control" aria-label="Username" v-model="event.title"
                                aria-describedby="basic-addon1">
                        </div>
                        <div class="input-group mb-3">
                            <span class="input-group-text" id="basic-addon1" style="min-width: 135px;">Start tidspunkt</span>
                            <input type="datetime-local" class="form-control" aria-label="Username"
                                aria-describedby="basic-addon1" v-model="event.startDate">
                            <!-- <span class="input-group-text" id="basic-addon1">Slut tidspunkt (optimal)</span>
                            <input type="datetime-local" class="form-control" aria-label="Username"
                                aria-describedby="basic-addon1" v-model="event.endDate"> -->
                        </div>
                        <div class="input-group">
                            <span class="input-group-text" style="min-width: 135px;">Beskrivelse</span>
                            <textarea class="form-control" v-model="event.description"></textarea>
                        </div>
                        <CalendarEvent :id="event.id" :title="event.title || 'Title'" :startDate="event.startDate" :endDate="event.endDate" :description="event.description || '...'"></CalendarEvent>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Luk</button>
                        <button v-if="!this.shouldUpdate" type="button" class="btn btn-warning" @click="addEvent()">Tilføj</button>
                        <button v-else type="button" class="btn btn-warning" @click="updateEvent()">Ændre</button>
                    </div>
                </div>
            </div>
        </div>
        <div class="w-75 event-container">
            <div class="d-flex justify-content-between">
                <div class="dropdown">
                    <span class="input-group-text filter-span"><i class="bi bi-filter-circle"></i></span>
                    <!-- <div class="filter-items ms-4">
                        <div class="filter-item m-1 btn d-flex">Seneste</div>
                    </div> -->
                </div>
                <div class="input-group me-4 search-input">
                    <span class="input-group-text search-span"><i class="bi bi-search"></i></span>
                    <input class="form-control" placeholder="søg efter begivenheder" @input="searchForEvents($event)">
                </div>
            </div>
        </div>
        <div class="d-flex justify-content-center mt-5">
            <template v-if="hasLoaded">
                <template v-if="hasEvents">
                    <template v-for="event in events" :key="event.id">
                        <CalendarEvent :id="event.id" :title="event.title" :startDate="event.startDate" :endDate="event.endDate" :description="event.description" @deleteEvent="deleteEvent(event.id, event.title)" @openUpdateModal="openUpdateModal(event.id)"></CalendarEvent>
                    </template>
                </template>
                <div v-else class="">
                    Ingen kommende begivenheder ...
                </div>
            </template>
            <div class="spinner-border position-absolute" role="status" v-else>
                <span class="visually-hidden">Loading...</span>
            </div>
        </div>
    </div>
</template>
<script>
    import { mapStores } from 'pinia'
    import { authStore, eventStore } from '@/store';
    import CalendarEvent from '@/components/CalendarEvent.vue'
    import { toastSuccess, toastError, toastWarning } from '../services/toasty.js'

    export default {
        name: "calender-route",
        components: {
            CalendarEvent
        },

        data() {
            return {
                hasLoaded: false,
                shouldUpdate: false,
                event: {
                    title: "",
                    startDate: null,
                    endDate: null,
                    description: "",
                    id: ""
                },
            }
        },

        computed: {
            ...mapStores(authStore, eventStore),

            user() {
                return this.authStore.user
            },

            events() {
                return this.eventStore.events
            },

            hasEvents() {
                return this.events.length > 0
            },
        },

        methods: {
            async getEvent(docId) {
                return await this.eventStore.getEvent({ docId })
            },

            async addEvent() {
                const {
                    title,
                    startDate,
                    description
                } = this.event
                try {
                    await this.eventStore.addEvent({ title, startDate, description })    
                    this.clearInputFields()
                    toastSuccess("Begivenhed tilføjet")
                } catch(e) {
                    toastError('Der skete en fejl')
                }
            },

            deleteEvent(docId, eventTitle) {
                if (confirm("Er du sikker på at du vil slette " + eventTitle + "?")) {
                    try {
                        this.eventStore.deleteEvent({ docId })
                        toastWarning('Begivenhed slettet')
                    } catch(e) {
                        toastError('Der skete en fejl')
                    }
                }
            },

            updateEvent() {
                const {
                    id,
                    title,
                    startDate,
                    description
                } = this.event
                try {
                    this.eventStore.updateEvent({ docId: id, title, startDate, description })
                    this.clearInputFields()
                    toastSuccess("Begivenhed opdateret")
                } catch(e) {
                    toastError('Der skete en fejl')
                }
            },  

            openAddEventModal() {
                this.shouldUpdate = false
                this.clearInputFields()
            },

            async openUpdateModal(docId) {
                this.shouldUpdate = true
                const currEvent = await this.getEvent(docId)
                this.event =  { ...currEvent, id: docId }
            },

            clearInputFields() {
                this.event = {
                    id: "",
                    title: "",
                    startDate: null,
                    endDate: null,
                    description: ""
                }
            },

            searchForEvents(e) {
                const needle = e.target.value
                const searchKeyword = "title"
                this.eventStore.searchForEvents({ needle, searchKeyword })
            }
        },

        created() {
            this.eventStore.fetchEvents().then(() => this.hasLoaded = true)
        }
    }
</script>
<style scoped>
    i {
        margin-right: 10px;
    }

    i:hover {
        opacity: 0.5;
        cursor: pointer;
    }

    textarea {
        height: 200px;
    }

    .search-span {
        padding-right: 2px;
    }

    .search-input {
        width: 35%;
        text-overflow: ellipsis;
    }

    .form-control {
        font-size: small;
    }

    .dropdown {
        position: relative;
        display: inline-block;
    }

    .filter-span {
        padding-right: 2px;
        margin-left: 1.5rem;

    }

    .filter-items {
        display: none;
        position: absolute;
        background-color: #f1f1f1;
        min-width: 160px;
        box-shadow: 0px 8px 16px 0px rgba(0, 0, 0, 0.2);
        z-index: 1;
    }

    .filter-item:hover {
        opacity: 0.5;
    }

    .dropdown:hover .filter-items {
        display: block;
    }

    /* media-q for smaller screen sizes */
    @media only screen and (max-width: 1140px) {
        .search-input {
            width: 50%;
            text-overflow: ellipsis;
        }

        .event-container {
            width: 100% !important;
        }
    }

    @media only screen and (max-width: 720px) {
        .search-input {
            width: 100%;
            text-overflow: ellipsis;
        }

        .event-container {
            width: 100% !important;
        }
    }
</style>