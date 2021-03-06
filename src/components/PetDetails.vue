<template>
  <div id="petDetails" class="container">
    <div class="loader" v-if="loading"></div>
    <div class="errors" v-if="errors.length">
      <h2 v-for="error of errors">{{ error.message }}</h2>
    </div>
    <div id="pet-container" v-else-if="!loading">
      <div id="swiper-wrapper col-12">
        <swiper :options="swiperOption">
          <swiper-slide v-for="img in this.galleryLength" :key="img.id">
            <div class="slide-content" @click="openModal"><img class="img-fluid" :src="concatUrl(img)" @error="imageHandler" alt=""></div>
          </swiper-slide>
          <div class="swiper-pagination" slot="pagination"></div>
          <div class="swiper-button-prev" slot="button-prev"></div>
          <div class="swiper-button-next" slot="button-next"></div>
        </swiper>
      </div>
      <Modal :pet="petDetails" v-if="showModal" @close="showModal = false" class="modal col-sm-12 col-lg-12"></Modal>
      <div class="petInfo">
        <ul class="list-group pet-ul col-sm-12">
          <li class="pet-li row">
            <div class="center-span col-sm-12 col-lg-6">
              <h6>{{petDetails.name}}</h6>
              <span class="li-span-name">{{$t('petDetails.name')}}</span>
            </div>
          </li>
          <li class="pet-li row">
            <div class="center-span col-sm-12 col-lg-6">
              <h6>{{petDetails.type}}</h6>
              <span class="li-span-dog">{{$t('petDetails.type')}}</span>
            </div>
            <div class="col-sm-12 col-lg-6 divider">
              <h6 class="center-span">{{petDetails.sex}}</h6>
              <span class="center-span li-span-sex">{{$t('petDetails.sex')}}</span>
            </div>
          </li>
          <li v-show="authenticated" class="pet-li row">
            <div class="center-span col-sm-12 col-lg-6">
              <h6>{{convertTimeStamp}}</h6>
              <span class="li-span-calendar center-span">{{$t('petDetails.creationDate')}}</span>
            </div>
          </li>
          <li class="pet-li row">
            <div class="description-container">
              <p>{{petDetails.description}}</p>
            </div>
          </li>
        </ul>
      </div>
      <button class="helpLink btn" :disabled="showTransfer" @click.once="showTransfer = true">{{$t('common.button.help')}}</button>
      <div class="col-sm-12 col-lg-12" v-show="showTransfer">
        <h6> {{petDetails.name}} {{$t('petDetails.info')}} </h6>
        <h5>{{formattedAccountNumber}}</h5>
      </div>
      <hr>
      <comments v-if="authenticated" :petId="petDetails.id"></comments>
    </div>
  </div>
</template>
<script>
  import transfer from './Transfer.vue'
  import comments from './Comments.vue'
  import { swiper, swiperSlide } from 'vue-awesome-swiper'
  import moment from 'moment'
  import auth from '../auth.js'
  export default {
    data () {
      return {
        showModal: false,
        showTransfer: false,
        authenticated: false,
        id: this.$route.params.id,
        petDetails: {},
        bankAccount: {
          number: ''
        },
        errors: [],
        swiperOption: {
          nextButton: '.swiper-button-next',
          prevButton: '.swiper-button-prev',
          pagination: '.swiper-pagination',
          paginationType: 'fraction'
        },
        loading: true
      }
    },
    components: {
      swiper,
      swiperSlide,
      transfer,
      comments
    },
    mounted () {
      this.fetchData()
      this.getTransferData()
    },
    methods: {
      openModal () {
        this.showModal = true
      },
      fetchData () {
        this.$http.get(this.apiUrl + 'pets/' + this.id)
          .then(response => {
            this.petDetails = {...response.data}
            this.authenticated = auth.user.authenticated
            this.loading = false
          })
          .catch(error => {
            this.errors.push(error)
            this.loading = false
          })
      },
      getTransferData () {
        this.$http.get(this.apiUrl + 'organization/info')
          .then(response => {
            this.bankAccount.number = response.data.bankAccount.number
            this.loading = false
          })
          .catch(error => {
            this.errors.push(error)
            this.loading = false
          })
      },
      imageHandler () {
        if (this.petDetails.type === 'dog' || this.petDetails.type === 'DOG') {
          this.petDetails.imageUrl = require('../assets/default_photo_dog.svg')
          return this.petDetails.imageUrl
        } else {
          this.petDetails.imageUrl = require('../assets/default_photo_cat.svg')
          return this.petDetails.imageUrl
        }
      },
      concatUrl (img) {
        if (this.petDetails.gallery === '' || this.petDetails.gallery === null || this.petDetails.gallery === undefined) {
          return this.imageHandler()
        } else {
          return this.apiUrl.substr(0, this.apiUrl.length - 4) + img.fileUrl
        }
      }
    },
    computed: {
      convertTimeStamp () {
        const date = moment(this.petDetails.created).locale(this.$t('common.code'))
        return date.format(this.$t('common.dateFormat'))
      },
      formattedAccountNumber () {
        const accountNumber = this.bankAccount.number
        return `${accountNumber.substr(0, 2)} ${accountNumber.substr(2, 4)} ${accountNumber.substr(6, 4)} ${accountNumber.substr(10, 4)} ${accountNumber.substr(14, 4)} ${accountNumber.substr(18, 4)} ${accountNumber.substr(22, 4)}`
      },
      galleryLength () {
        const ifEmpty = 1
        if (this.petDetails.gallery === null || this.petDetails.gallery === undefined || this.petDetails.gallery === 0) {
          return ifEmpty
        } else {
          return this.petDetails.gallery
        }
      }
    }
  }
</script>
<style lang="sass" scoped>
  @import "../assets/styles/loader.css"
  @import "../assets/styles/variables.sass"
  #petDetails
    margin-top: 8em

  .petInfo
    margin: 2em auto
    width: 96em
    display: block

  .description-container
    margin: 0 auto
    width: 96em
    display: block

  .img-fluid
    height: 100%

  .pet-ul
    width: 100%
    border-top: 0
    padding: 0
    margin: 3.0em 0 3.0em 0
    list-style-type: none

  .pet-li
    width: 100%
    padding: 1.5em 0
    margin: 0
    border: 0
    border-bottom: 1px solid lightgray
    font-weight: bold
    display: flex
    flex-direction: row

  .helpLink
    font: $font-stack
    font-weight: bold
    margin: 2em auto
    padding: .8em
    width: 20em
    height: 3em
    display: flex
    justify-content: center
    background-color: $orange
    border-radius: 0.4em

  .disabled
    background-color: $silver
    &:hover
      background-color: $silver
  .sex-wrapper
    width: 50%
    float: right
    margin-left: 35vw

  .center-span
    flex-direction: column
    display: flex
    text-align: left

  .li-span-dog
    color: $black
    margin-top: .5em

  .li-span-sex
    color: $black
    margin-top: .9em

  .li-span-name
    color: $black
    margin-top: .5em
    float: left

  .li-span-calendar
    color: $black
    margin-top: .5em

  .float-left-span
    float: left
    margin-left: .5em

  .slide-content
    width: 96em
    height: 70em
    margin: 0 auto

  .imgSlider
    width: 96em
    height: 70em
    background-size: cover
    background-position: center

  .swiper-slide
    background-position: center
    background-size: cover

  .swiper-pagination
    background-color: #000000
    opacity: 0.7
    font: $font-stack

  .swiper-pagination-fraction
    bottom: 0
    color: $white
    font-size: 1.8em !important
    padding: 1em

  #swiper-wrapper
    width: 96em
    margin: 0 auto

  /* MEDIA QUERIES */
  @media only screen and (max-width: 576px)
    #swiper-wrapper
      width: 100%
      margin: 0 auto

    .slide-content
      width: 34.5em
      height: 28.7em
      margin: 0 auto

    .petInfo
      margin: 0 auto
      width: 100%
      display: block

    .pet-li
      margin: 0 auto
      width: 100%

    .pet-ul
      margin: 1.0em 0 1.0em 0
      list-style-type: none

    .sex-wrapper
      border-top: 1px black solid
      width: 40vw

    .divider
      border-top: 1px solid lightgray
      padding-top: 1.4em

    .li-span-dog
      padding-bottom: 1.2em

</style>
