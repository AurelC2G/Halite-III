<template>
    <div class="logged-in">
      <div id="profile" class="profile-container">
        <a v-on:click.stop="slide_profile">
          <profile-image
            height="40"
            :username="username"
            :profileImage="profile_image"
          />
          <span class="user-name">{{username}}</span>
          <i class="fa fa-sort-down"></i>
          <ul class="nav profile-nav" v-if="!isCreateAccount">
            <li class="profile-nav-item"><a v-on:click="gaData('account','click-view-profile','account-flow')" href="/user?me"><span>view profile</span><i class="line line-bottom"></i></a></li>
            <li class="profile-nav-item"><a v-on:click="gaData('account','click-edit-profile','account-flow')" href="/user/edit-user"><span>edit profile</span><i class="line line-bottom"></i></a></li>
            <li class="profile-nav-item"><a v-on:click="gaData('account','click-edit-settings','account-flow')" href="/user/settings"><span>settings</span><i class="line line-bottom"></i></a></li>
            <li class="profile-nav-item"><a v-on:click.stop.prevent="sign_out"><span>sign out</span><i class="line line-bottom"></i></a></li>
          </ul>
          <ul class="nav profile-nav" v-else>
            <li class="profile-nav-item"><a v-on:click.stop.prevent="sign_out"><span>sign out</span><i class="line line-bottom"></i></a></li>
          </ul>
        </a>
      </div>
    </div>
</template>

<script>
    import * as api from '../api'
import * as utils from '../utils'

export default {
      name: 'user-profile-bar',
      props: ['baseUrl'],
      data: function () {
        const me = api.me_cached()
        const isMobile = window.mobileAndTabletcheck()
        if (me) {
          return {
            username: me.username,
            profile_image: me.profile_image,
            isMobile: isMobile,
            isCreateAccount: window.location.pathname == '/create-account'
          }
        }
        return {
          username: '',
          profile_image: null,
          isMobile: isMobile,
          isCreateAccount: false // window.location.pathname == '/create-account'
        }
      },
      mounted: function () {
        api.me().then((user) => {
          this.username = user.username
          this.profile_image = user.profile_image_key
          $('profile').addClass('container-loaded')
          $('submitbutton').addClass('container-loaded')
        })
        document.addEventListener('click', (e) => {
          if (!this.$el.contains(e.target)) {
            this.leave_profile()
          }
        })
      },
      methods: {
        slide_profile: function (e) {
          const currentTarget = e.currentTarget
          $(currentTarget).find('ul').slideToggle()
        },
        leave_profile: function (e) {
          let currentTarget = e ? e.currentTarget : $('.profile-container ul')
          $(currentTarget).stop().slideUp()
        },
        gaData: function (category, action, label) {
          utils.gaEvent(category, action, label)
        },
        sign_out: function (e) {
          this.gaData('account', 'click-sign-out', 'account-flow')
          api.logout().then((res) => {
            window.location.replace('/')
          })
        }
      }
    }
</script>

<style lang="scss" scoped>
.profile-nav{
  min-width: 230px;
  li.profile-nav-item{
    background: #001c4c;
    a{
      padding: 13px 15px;
      font-size: 14px;
      color: #97DDFF;
      font-weight:400;
      opacity: .6;
      display: block;
      border-left: 2px solid transparent;
      &:hover {
        color: #91DBFF;
        text-shadow: 0 -1px 13px 0 #00ABFF;
        border-left: 2px solid #91DBFF;
        background-color: transparent;
        opacity: 1;
        &:before {
          opacity: 0.4;
          background-image: linear-gradient(269deg, rgba(117,182,219,0) 0%, #97DDFF 100%);
          content: "";
          display: block;
          position: absolute;
          left: 0;
          top: 0;
          width: 100%;
          height: 100%;
        }
      }
    }
  }
}
</style>
