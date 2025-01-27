<script>
    import axios from 'axios';
    import { store } from '../store.js';
    import AppLoader from '../components/AppLoader.vue';

    export default {
        name: 'SingleDoctor',
        components: {
            AppLoader
        },
        data() {
            return {
                store,
                doctor: null,
                loading: false,
                submit: false,
                userName: '',
                userEmail: '',
                userMessage: '',
                userObject: '',
                errors: {},
                userNameReview: '',
                userMessageReview: '',
                submitReview: false,
                userTc: '',
                userTcReview: '',
                selectedRating: '', // Aggiungi questa linea
                doctorID: '',
                ratingID: '',
                messageSuccess: false,
                reviewSuccess: false,
                minRating: 0,
            }
        },
        methods: {
            getSingleDoctor() {
                axios.get(`${this.store.apiUrl}/api/profiles/${this.$route.params.slug}`)
                .then((response) => {
                    this.doctorID = response.data.profile.id;
                    this.doctor = response.data.profile;
                    this.loading = true;
                })
            },
            sendMessage() {
                const userMessage = {
                    name: this.userName,
                    email: this.userEmail,
                    message: this.userMessage,
                    object: this.userObject,
                    accepted_tc: this.userTc,
                    profile_id: this.doctorID,
                };

                this.submit = true;

                axios.post(`${this.store.apiUrl}/api/messages`, userMessage)
                .then(response => {
                    this.messageSuccess = response.data.success;
                    if(response.data.success) {
                        this.errors = {},
                        this.userName = '';
                        this.userEmail = '';
                        this.userMessage = '';
                        this.userObject = '';
                        this.userTc = '';
                    } else {
                        this.errors = response.data.errors;
                    }
                    this.submit = false;
                })
            },
            contactUsAgain(){
                this.messageSuccess = false;
            },
            reviewUsAgain(){
                this.reviewSuccess = false;
            },
            sendReviewAndRating() {
                const userRate = {
                    score: this.selectedRating, // Cambia userRate con selectedRating
                };
                axios.post(`${this.store.apiUrl}/api/ratings`, userRate)
                .then(response => {
                    this.reviewSuccess = response.data.success;
                    if(response.data.success) { 
                        console.log('response' , response.data.data.id);
                        this.ratingID = response.data.data.id;
                        this.errors = {},
                        this.selectedRating = ''; // Resetta selectedRating
                    } else {
                        this.errors = response.data.errors;
                    }
                    const userReview = {
                        name: this.userNameReview,
                        description: this.userMessageReview,
                        profile_id: this.doctorID,
                        rating_id: this.ratingID,
                    };
    
                    this.submitReview = true;
    
                    axios.post(`${this.store.apiUrl}/api/reviews`, userReview)
                        .then(response => {
                            if(response.data.success) {
                                this.errors = {},
                                this.userNameReview = '';
                                this.userMessageReview = '';
                                this.ratingID = '';
                            } else {
                                this.errors = response.data.errors;
                            }
                            this.submitReview = false;
                        })
                        
                })
            },
            highlightStars(rating) {
                const stars = document.querySelectorAll('.rating label');
                stars.forEach((star, index) => {
                    if (index <= (5 - rating) && rating !== 0) {
                        star.style.color = '#f5b301';
                    } else {
                        star.style.color = '#ddd';
                    }
                });
            }
        },
        watch: {
            minRating: 'filterDoctors',
            searchQuery: 'filterDoctors',
            filterByRating: 'filterDoctors'
        },

        mounted() {
            this.getSingleDoctor();
        },
    }
</script>


<template>
    <section class="py-4">
        <div v-if="!loading" class="container py-3">
            <AppLoader></AppLoader>
        </div>
        <div v-else>
            <div class="container py-3">
                <div class="wrapper d-flex align-items-center justify-content-between">
                    <h1>{{ doctor.spec_name }}</h1>
                    <router-link :to="{ name: 'our-doctors' }" class="btn btn-brand">Ritorna</router-link>
                </div>
                <div class="doctor-wrapper card my-4 p-3">
                    <div class="image-center">
                        <img class="profile-photo mb-3" v-if="doctor.photo" :src="`http://127.0.0.1:8000/storage/${doctor.photo}`" :alt="doctor.user_name">
                    </div>
                    <p><strong>Nome:</strong> {{ doctor.user.name }}</p>
                    <p><strong>Email:</strong> {{ doctor.user.email }}</p>
                    <p v-if="doctor.specialisations.length > 0" class="card-text"><strong>Specializzazioni:</strong>
                        <span v-for="(specialisation, index) in doctor.specialisations" :key="specialisation.id">
                            {{ specialisation.name }}<span v-if="index < doctor.specialisations.length - 1">,</span> &nbsp;
                        </span>
                    </p>
                    <p><strong>Performance:</strong> {{ doctor.performance }}</p>
                    <p><strong>Telefono:</strong> {{ doctor.telephone_number }}</p>
                    <p><strong>Biografia:</strong> {{ doctor.bio }}</p>
                    <p><strong>Curricum Vitae:</strong></p>
                        <div><img class="profile-photo mb-3" v-if="doctor.photo" :src="`http://127.0.0.1:8000/storage/${doctor.curriculum_vitae}`" :alt="doctor.user_name"></div>
                </div>
            </div>
            
    
            <!-- Form invio messaggio al dottore -->
            <div v-if="messageSuccess" class="alert container alert-success d-flex align-items-center justify-content-between" role="alert">
                <p>Grazie per averci contattato, ti ricontatteremo il prima possibile!</p>
                <button @click="contactUsAgain" class="btn btn-primary btn-brand">Scrivici ancora!</button>
            </div>
            <div v-else class="contact container p-2 mb-4">
                <h2>Contatta il tuo Medico</h2>
                <div class="danger mb-2">* campi obbligatori</div>
                <form @submit.prevent="sendMessage">
    
                    <!-- Input user name -->
                    <div class="mb-3">
                        <label class="form-check-label me-2" for="name"><span class="text-danger">*</span> Nome</label>
                        <div></div>
                        <input  type="text" name="name" id="name" v-model="userName">
                    </div>
                    <div v-if="errors.name">
                        <div v-for="error in errors.name" class="alert alert-danger" role="alert">
                            {{ error }}
                        </div>
                    </div>
                    <!-- /Input user name -->
    
                    <!-- Input object -->
                    <div class="mb-3">
                        <label class="form-check-label me-2 fw-bold" for="object"><span class="text-danger">*</span> Oggetto</label>
                        <div></div>
                        <input type="text" name="object" id="object" v-model="userObject">
                    </div>
                    <div v-if="errors.name">
                        <div v-for="error in errors.name" class="alert alert-danger" role="alert">
                            {{ error }}
                        </div>
                    </div>
                    <!-- /Input object -->
    
                    <!-- Input user email -->
                    <div class="mb-3">
                        <label for="email" class="form-check-label me-2 fw-bold"><span class="text-danger">*</span> Email</label>
                        <div></div>
                        <input type="email" id="email" aria-describedby="emailHelp" name="email" v-model="userEmail">
                    </div>
                    <div v-if="errors.email">
                        <div v-for="error in errors.email" class="alert alert-danger" role="alert">
                            {{ error }}
                        </div>
                    </div>
                    <!-- /Input user email -->
    
                    <!-- Input user message -->
                    <div class="mb-3">
                        <label for="message" class="form-check-label me-2 fw-bold"><span class="text-danger">*</span> Chiedi al medico:</label>
                        <textarea name="message" rows="10" id="message" v-model="userMessage"></textarea>
                    </div>
                    <div v-if="errors.message">
                        <div v-for="error in errors.message" class="alert alert-danger" role="alert">
                            {{ error }}
                        </div>
                    </div>
                    <!-- /Input user message -->
    
                    <!-- Accept TC -->
                    <div class="mb-3 form-check">
                        <label class="form-check-label" for="accepted_tc">Accetta termini e condizioni <router-link :to="{name: 'terms-and-conditions-messages'}" target="_blank">Mostra termini e condizioni</router-link></label>
                        <input type="checkbox" class="form-check-input" name="accepted_tc" id="accepted_tc" v-model="userTc">
                    </div>
                    <div v-if="errors.accepted_tc">
                        <div v-for="error in errors.accepted_tc" class="alert alert-danger" role="alert">
                            {{ error }}
                        </div>
                    </div>
                    <!-- /Accept TC -->
    
                    <button :disable="submit" type="submit" class="btn btn-brand">{{ submit ? 'Invio...' : 'Invia'}}</button>
                </form>
            </div>
            <!-- /Form invio messaggio al dottore -->
    
            <!-- Form recensione dottore -->
            <div v-if="reviewSuccess" class="alert container alert-success d-flex align-items-center justify-content-between" role="alert">
                <p>Grazie per averci dato il tuo feedback, a presto!</p>
                <button @click="reviewUsAgain" class="btn btn-primary btn-brand">Facci sapere altro!</button>
            </div>
            <div v-else class="review-wrapper container p-2">
                <h2>Scrivi una recensione</h2>
                <div class="danger mb-2"><span class="text-danger">*</span> campi obbligatori</div>
                <form @submit.prevent="sendReviewAndRating">
                    <!-- Input user name -->
                    <div class="mb-3">
                        <label class="form-check-label me-2 fw-bold" for="name-review">Nome</label>
                        <div></div>
                        <input type="text" name="name" id="name-review" v-model="userNameReview">
                    </div>
                    <div v-if="errors.name">
                        <div v-for="error in errors.name" class="alert alert-danger" role="alert">
                            {{ error }}
                        </div>
                    </div>
                
                    <div class="mb-1">
                        <label for="description-review" class="form-check-label me-2 fw-bold"><span class="text-danger">*</span> Lascia la tua recensione:</label>
                        <textarea name="description" rows="10" id="description-review" v-model="userMessageReview"></textarea>
                        
                    </div>
                        
                    <div v-if="errors.description">
                        <div v-for="error in errors.description" class="alert alert-danger" role="alert">
                            {{ error }}
                        </div>
                    </div>
       
                        <option selected class="fw-bold">Valuta il medico</option>
                        <div class="rating mb-3" id="rating">
                                <input type="radio" id="star1" name="rating" value="1" v-model="selectedRating" @change="highlightStars(5)">
                                <label for="star1" class="me-2">★</label>
                                <input type="radio" id="star2" name="rating" value="2" v-model="selectedRating" @change="highlightStars(4)">
                                <label for="star2" class="me-2">★</label>
                                <input type="radio" id="star3" name="rating" value="3" v-model="selectedRating" @change="highlightStars(3)">
                                <label for="star3" class="me-2">★</label>
                                <input type="radio" id="star4" name="rating" value="4" v-model="selectedRating" @change="highlightStars(2)">
                                <label for="star4" class="me-2">★</label>
                                <input type="radio" id="star5" name="rating" value="5" v-model="selectedRating" @change="highlightStars(1)">
                                <label for="star5" class="me-2">★</label>
                        </div>        


                    <!-- Input user description -->
    
                    <!-- User rating -->

                    <!-- /User rating -->
    
                    <!-- Accept TC -->
                    <!-- <div class="mb-3 form-check">
                        <input type="checkbox" class="form-check-input" id="accepted_tc" v-model="userTc">
                        <label class="form-check-label" for="accepted_tc">Accetta termini e condizioni <router-link :to="{name: 'terms-and-conditions-reviews'}" target="_blank">Mostra termini e condizioni</router-link></label>
                    </div>
                    <div v-if="errors.accepted_tc">
                        <div v-for="error in errors.accepted_tc" class="alert alert-danger" role="alert">
                            {{ error }}
                        </div>
                    </div> -->
                    <!-- /Accept TC -->
    
                    <button :disable="submitReview" type="submit" class="btn btn-brand">{{ submitReview ? 'Invio...' : 'Invia'}}</button>
                </form>
                                    
            </div>
            
            <!-- /Form recensione dottore -->
        </div>
        <footer>
        <div class="container">
            <p>© 2024 BDoctors. Tutti i diritti riservati. Le informazioni fornite su questo sito, comprese le prenotazioni delle visite mediche, i messaggi al medico e le recensioni, sono a scopo informativo e non sostituiscono il consiglio medico professionale.</p>
        </div>
    </footer>
    </section>

</template>

<style lang="scss" scoped>
    @use '../style/partials/variables' as *;

    section {
        background-color: $primary-color;
        color: white;
        .btn-brand {
            color: $primary-color;
            background-color: $secondary-color;
        }
        .image-center {
            display: flex;
            justify-content: center;
        }
        .profile-photo {
            max-height: 500px;
            max-width: 500px;
            object-fit: cover;
        }
        .alert-success > p{
            margin-bottom: 0;
        }
        p {
            strong {
                color: $primary-color;
            }
        }
        .ms-label{
            display: block;
        }
        textarea{
            width: 100%;
        }
        .contact,
        .review-wrapper{
            border: 2px solid white;
            border-radius: 10px;
        }
        .danger{
            color:red;
        }
    }
    .rating {
    width: fit-content;
}

.rating > input {
    display: none;
}

.rating > label {
    font-size: 2rem;
    color: #ddd;
    cursor: pointer;
    padding: 0 0.1rem;
}

.rating > input:checked ~ label {
    color: #f5b301;
}

.rating > input:checked ~ label ~ label {
    color: #ddd;
}

.rating > input:focus ~ label {
    color: #f5b301;
}

footer {
        color: white;
        text-align: center;
        width: 50%;
        padding: 100px 0 10px 0;
        margin: 0 auto;
        p {
            border-top: 2px solid rgba(255, 255, 255, 0.522);
        }
        @media screen and (max-width:800px){
        width: 90%;
}
    }
</style>