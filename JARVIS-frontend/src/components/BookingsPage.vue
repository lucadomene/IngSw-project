<script setup lang="ts">
  import {useBookingStore} from '@/stores/bookingsStore';
  import {useUserStore} from '@/stores/userStore';
  import {storeToRefs} from 'pinia';
  import axios, { type AxiosResponse } from 'axios'
  import {Booking} from '@/types'
  import {ref} from "vue";

  const bookingStore = useBookingStore();
  const userStore = useUserStore();

  const {dataBooking} = storeToRefs(bookingStore);
  const {isDataBookingLoaded} = storeToRefs(bookingStore);
  const {dataUser} = storeToRefs(userStore);

  const popupTitle = ref<string>("placeholder title");
  const popupMessage = ref<string>("placeholder message");

  const modal = ref<HTMLDialogElement | null>(null);

  function useDialog(title:string, message:string) {
    popupTitle.value = title;
    popupMessage.value = message;
    modal.value!.showModal();
  }

  function closeDialog(){
    modal.value!.close();
  }

  async function deleteBooking(id: number){
    await axios.get<Booking>("/api/booking/del?id=" + id).then((response: AxiosResponse<Booking>) => {
      console.log("Risposta da Axios: ", response);
      console.log("Prenotazione cancellata con successo");
      for(let i = 0; i < dataBooking.value.length; i++){
        if(dataBooking.value[i].id == id){
          console.log(dataBooking.value[i]);
          dataBooking.value.splice(i, 1);
          break;
        }
      }
      bookingStore.updateData();
    }).catch(error => {
      useDialog("Errore!", error.message);
    });
  }
</script>

<template>
  <title>Prenotazioni</title>
  <h1 v-if="dataUser" class="first-content title" style="text-align: center;">Prenotazioni dell'utente: {{dataUser.codice_fiscale}}</h1>
  <div v-if="isDataBookingLoaded">
    <div v-if="dataBooking.length > 0">
      <transition-group name="list" tag="ul">
        <ul id="booking-info" v-for="booking in dataBooking" :key="booking.id">
          <li>
            <h1>{{booking.venue.name}}</h1>
            {{booking.date}} dalle ore {{booking.duration.start}} alle ore {{booking.duration.end}} <br>
            <strong>Indirizzo:</strong> {{booking.venue.address}} <br>
            <strong>Capacità massima:</strong> {{booking.venue.max_capacity}} <br>
            <strong>Costo:</strong> {{booking.venue.rent_cost}}€
            <h2 v-if="booking.personnel.length > 0">Personale assunto:</h2>
            <div v-for="personnel in booking.personnel" :key="personnel.name">
              {{personnel.name}} - {{personnel.hourly_cost}}€ all'ora
            </div>
            <div style="text-align:right">
              <button @click="deleteBooking(booking.id)" id="deleteBookingButton">
                Elimina
                <img src="/red-trash.png" alt="image of a trash bin">
              </button>
            </div>
          </li>
        </ul>
      </transition-group>
    </div>
    <div v-else class="container error-container">
      <img src="/ErrorImage.png" alt="Error image" class="image"/>
    </div>
  </div>
  <div v-else class="loader-container">
    <div class="dotsLoader"/>
    <div class="loader"/>
  </div>
  <dialog ref="modal" id="modal">
    <h2>{{popupTitle}}</h2>
    <p>{{popupMessage}}</p>
    <button class="button" id="close-button" @click="closeDialog">Chiudi</button>
  </dialog>
</template>

<style scoped>

  .error-container{
    border: solid var(--highlight-color) 3px;
    display:flex;
    justify-content: center;
    align-items: center;
    margin: 0 auto;
    height: 70vh;
    width: 100%;
    animation: fade-up 0.5s;
  }

  .image {
    max-width: 50%;
    max-height: 55%;
    object-fit: contain;
  }

  #booking-info {
	  border: solid var(--highlight-color) 3px;
    padding: 10px;
    margin: 10px 0;
    animation: fade-up 0.5s;
    list-style-type: none;
  }

  #booking-info strong {
    font-weight: bold;
  }

  #booking-info h1 {
    color: hsla(160, 100%, 37%, 1);
    font-weight: bold;
    padding: 0;
  }

  #booking-info h2 {
    font-size: 1.4em;
    margin-top: 5px;
  }

  #deleteBookingButton{
    font-weight: 700;
  }

  .list-move,
  .list-enter-active,
  .list-leave-active {
    transition: all 0.9s ease;
  }

  .list-enter-from,
  .list-leave-to {
    opacity: 0;
    transform: translateX(30px);
  }

  .list-leave-active {
    position: absolute;
  }

  .loader-container {
    border: solid var(--highlight-color) 3px;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 70vh;
    width: 100%;
    margin: 0 auto;
    text-align: center;
  }

  .loader {
    width: fit-content;
    font-weight: 700;
    font-family: monospace;
    font-size: 30px;
    color :#0000;
    overflow: hidden;
    animation: l9 5s infinite cubic-bezier(0.3,1,0,1);
  }
  .loader:before {
    content:"Loading...";
  }
  @keyframes l9 {
    0%  {text-shadow: 0    0 #FFF, 11ch 0 #fa1c1c, 22ch 0 #11EDFFFF, 33ch 0 var(--highlight-color),44ch 0 var(--navbar-text-color)}
    25% {text-shadow:-11ch 0 #FFF,  0 0 #ec9d09, 11ch 0 #11EDFFFF, 22ch 0 var(--highlight-color),33ch 0 var(--navbar-text-color)}
    50% {text-shadow:-22ch 0 #FFF,-11ch 0 #ec9d09,  0 0 #11EDFFFF, 11ch 0 var(--highlight-color),22ch 0 var(--navbar-text-color)}
    75% {text-shadow:-33ch 0 #FFF,-22ch 0 #ec9d09,-11ch 0 #11EDFFFF,  0 0 var(--highlight-color),11ch 0 var(--navbar-text-color)}
    100%{text-shadow:-44ch 0 #FFF,-33ch 0 #ec9d09,-22ch 0 #11EDFFFF,-11ch 0 var(--highlight-color), 0 0 var(--navbar-text-color)}
  }

  .dotsLoader {
    color: var(--highlight-color);
    margin: 0 55px 0;
    width: 4px;
    aspect-ratio: 1;
    border-radius: 50%;
    box-shadow: 19px 0 0 7px, 38px 0 0 3px, 57px 0 0 0;
    transform: translateX(-38px);
    animation: l21 .5s infinite alternate linear;
  }
  @keyframes l21 {
    50%  {box-shadow: 19px 0 0 3px, 38px 0 0 7px, 57px 0 0 3px}
    100% {box-shadow: 19px 0 0 0  , 38px 0 0 3px, 57px 0 0 7px}
  }
</style>
