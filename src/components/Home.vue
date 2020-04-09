<template>
  <v-container>
    <div v-if="isLoading" class="loader"></div>
    <div v-if="!isLoading">
      <h1 class="text-center mb-4 mt-4">Company List</h1>
      <div class="d-flex flex-row justify-space-between filter-container">
        <v-text-field 
          label="Search..." 
          v-model="searchCompany"
          append-icon="mdi-magnify"
          class="col-10"
          solo
          hide-details
        ></v-text-field>
        <v-icon 
          @click="showBookmarked"
          :class="showBookmarkedList === true ? 'active' : ''"
          class="col-2"
        >mdi-bookmark</v-icon>
      </div>
      <v-row style="height: 400px">
        <div v-for="(item,index) in filteredCompanyList"
          :key="index" class="col-lg-4 col-md-6 col-sm-12">
          <v-card outlined class="ma-5">
            <div class="pa-0 card-container">
              <v-list-item-content 
                class="pa-5"
                style="cursor: pointer;" 
                @click="showCompanyDetail(item)"
              >
                <div class="mt-2 mb-2 d-flex flex-row justify-space-between">
                  <span class="card-container__group-name">{{ item.group }}</span>
                  <span class="card-container__booth">{{ item.booth }}</span>
                </div>
                <div class="card-container__company-name mb-2"><b>{{ item.name }}</b></div>
                <div class="card-container__description">{{ item.description }}</div>
              </v-list-item-content>
              <div class="card-container__actions">
                <v-icon 
                  @click="bookmarked(item.name)"
                  :class="item.bookmarked === true ? 'active' : ''"
                  class="mr-3"
                >mdi-bookmark</v-icon>
                <v-icon :class="item.messaged > 0 ? 'active' : ''">mdi-message</v-icon>
              </div>
            </div>
          </v-card>
        </div>
      </v-row>

      <v-dialog
        v-model="showDetail"
        color="transparent"
        width="80%"
      >
        <v-card 
          v-for="(item,index) in detailData"
          :key="index"
          class="company-detail__container"
        >
          <div class="company-detail__content">
            <img :src="item.logo" />
            <div>
              <h1 class="mb-2">{{ item.name }}</h1>
              <v-icon 
                @click="bookmarked(item.name)"
                :class="item.bookmarked === true ? 'active' : ''"
                class="bookmark-button"
              >mdi-bookmark</v-icon>
              <span>{{ item.group }}</span>
              <p class="mb-0 mt-2">{{ item.description }}</p>
              <v-btn 
                outlined
                class="message-button"
                @click="sendMessage(item.name)"
                v-if="!messageForm"
              >
                Send Message
              </v-btn>
              <div v-if="messageForm" class="ml-0">
                <v-form>
                  <v-text-field
                    v-model="email"
                    label="Email"
                    required
                    class="ml-0 mt-3"
                    hide-details
                  ></v-text-field>
                  <v-textarea
                    rows="2"
                    label="Message"
                    v-model="message"
                    class="ml-0 mt-3"
                    hide-details
                  ></v-textarea>
                  <v-btn 
                    outlined
                    class="message-button mt-4"
                    :disabled="!isCompleted"
                    @click="saveMessage(item.name)"
                  >Send Message</v-btn>
                </v-form>
              </div>
            </div>
          </div>
        </v-card>
      </v-dialog>
    </div>
  </v-container>
</template>

<script>
  import axios from 'axios'
  export default {
    data: () => ({
      searchCompany: '',
      companyList: [],
      filteredCompanyList: [],
      detailData: [],
      showBookmarkedList: false,
      isLoading: true,
      valid: true,
      showDetail: false,
      messageForm: false,
      validEmail: false,
      sendMessageTo: "",
      message: "",
      email: "",
      emailValidation: /.+@.+\..+/
    }),
    created() {
      axios.get(`https://api.jublia.com/buzz/v2/directory/search`, {
        params: {
          event_id : `1b59351267938da712d19d57889c7f565cab96406e27a874607b90492a2845232f233a2b72bb4ae006a79b53f95aef054935c50b64d6a2a03cfe5cc75cbcd5cd`,
          client_id: 45426
        }
      }, {} )
      .then(response => {
        let dataTemp = response.data.searchResult.map( item => {
          return {
            name : item.company_name,
            description: item.company_description,
            logo: item.logo,
            group: item.group,
            bookmarkValue: item.messaged,
            booth: item.booth,
            attributes: item.attributes,
            bookmarked: false,
            messaged: item.messaged,
            bookmarkNumber: item.bookmark
          }
        })
        this.companyList = dataTemp
        this.filteredCompanyList = dataTemp
        this.isLoading = false
      })
    },
    computed: {
      bookmarkedList() {
        return this.filteredCompanyList.filter( item => {
          if ( item.bookmarked === true) {
            return item
          }
        })
      },
      isCompleted() {
        if ( this.message !== '' && this.email !== '' && this.validEmail ) {
          return true
        } else {
          return false
        }
      }
    },
    watch: {
      searchCompany(){
        if ( this.searchCompany.length > 0 ) {
          let companyList = this.companyList.filter( item => {
            if ( item.name.toLowerCase().indexOf(this.searchCompany.toLowerCase()) > -1 ) {
              return item
            }
          })
          this.filteredCompanyList = companyList
        } else {
          this.filteredCompanyList = this.companyList
        }
      },
      email() {
        if ( this.emailValidation.test(this.email) ) {
          this.validEmail = true
        } else {
          this.validEmail = false
        }
      }
    },
    methods: {
      bookmarked( company ) {
        this.companyList.filter( item => {
          if ( item.name.toLowerCase().indexOf(company.toLowerCase()) > -1 ) {
            item.bookmarked = !item.bookmarked
          }
        })
      },
      showBookmarked() {
        this.showBookmarkedList = !this.showBookmarkedList
        this.showBookmarkedList === true ? this.filteredCompanyList = this.bookmarkedList : this.filteredCompanyList = this.companyList
      },
      showCompanyDetail( item ) {
        this.detailData = []
        this.showDetail = true
        this.detailData.push(item)
      },
      sendMessage( name ) {
        this.messageForm = true
        this.sendMessageTo = name
      },
      saveMessage( name ) {
        this.filteredCompanyList.filter( item => {
          if ( item.name.toLowerCase().indexOf(name.toLowerCase()) > -1 ) {
            return item.messaged = 1
          }
          this.showDetail = false
        })
      }
    }
  }
</script>

<style lang="scss" scoped>
.loader {
  border: 16px solid #f9fac3;
  border-top: 16px solid #FEDE00;
  border-radius: 50%;
  width: 120px;
  height: 120px;
  animation: spin 2s linear infinite;
  position: fixed;
  top: 40%;
  left: 45%;
  transform: translate(-50%, -50%);
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.active {
  color: #FEDE00 !important;
}

.filter-container {
  margin-left: 20px;
}

.card-container {
  height: 230px;
  &__group-name, &__company-name, &__booth {
    font-family: montserrat, sans-serif;
  }
  &__group-name, &__booth {
    color: #FEDE00;
    font-weight: lighter;
  }
  &__company-name {
    font-size: 20px;  
    line-height: 1;
  }
  &__description {
    position: relative;
    overflow: hidden;
    line-height: 1.2;
    height: 50px; 
    font-size: 14px;
    text-align: justify;
    padding-right: 1em;
    &:before {
      content: '...';
      position: absolute;
      right: 0;
      bottom: 0;
    }
    &:after {
      content: '';
      position: absolute;
      right: 0;
      width: 1em;
      height: 1em;
      margin-top: 0.2em;
    }
  }
  &__actions {
    position: absolute;
    bottom: 0;
    text-align: right;
    width: 100%;
    padding: 0 20px 15px 0;
  }
}

.company-detail {
  &__container {
    padding: 40px;
  }
  &__content {
    display: flex;
    flex-direction: row;
    align-items: center;
    div {
      margin-left: 24px;
    }
    img {
      height: auto;
      width: 30%;
    }
    h1 {
      font-family: montserrat, sans-serif;
      line-height: 1.2;
    }
    span {
      color: #fcc900;
    }
    .bookmark-button {
      position: absolute;
      right: 30px;
      top: 15px;
    }
    .message-button {
      margin-top: 16px;
      background: linear-gradient(to right,#ffa84c 0%,#ff7200 52%,#ffe44c 100%);
      background-size: 300% 200%;
      color: #fff;
    }
    .message-button:hover {
      background: linear-gradient(to right,#ffe44c 0%,#ff7200 52%,#ffa84c 100%);
      background-size: 300% 200%;
    }
  }
}

@media screen and (max-width: 361px){
  .loader {
    top: 45%;
    left: 35%;
  }
  .filter-container {
    margin-left: 20px;
    margin-right: 20px;
  }
  .company-detail {
    &__container {
      padding: 15px;
    }
    &__content {
      padding: 12px;
      display: flex;
      flex-direction: column; 
      img {
        height: auto;
        width: 70%;
      }
      div {
        margin: 12px 0;
      }
      .bookmark-button {
        right: 10px;
        top: 10px;
      }
      p {
        line-height: 1.2;
      }
    }
  }
}
</style>
