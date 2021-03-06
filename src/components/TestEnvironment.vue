<template>
   <div class="container">
   <div id="test-environment">
          <span class="test-env-label">Test environment</span>
          <div class="test-env-name" @click="openDialog('dialog1')">
            <div class="icon">
            <img :src="'https://raw.githubusercontent.com/cncf/artwork/1d4e7cf3b60af40e008b2e2413f7a2d1ff784b52/kubernetes/icon/color/kubernetes-icon-noborder-color.png'" />
            </div>
            <div>
              Kubernetes &mdash; 
             {{ReleaseType(releaseType)}} {{ ReleaseTag(this.$props.project, releaseType) }}
            </div>
          </div>
          <div class="test-env-details">
              <div class="stage">Bare Metal (Packet)</div>
              <StatusLabel :url="ReleaseURL(this.$props.project, releaseType)" 
              :label="ReleaseStatus(this.$props.project, releaseType)"
              :branch="'stable'"
             :class="ReleaseStatus(this.$props.project, releaseType)"/>
          </div>
       </div>
    <div id="test-environment-full">
          <span class="test-env-label">Test environment</span>
          <div class="test-env-name" v-on:click="gotoProjectURL()">
            <div class="icon">
            <img :src="'https://raw.githubusercontent.com/cncf/artwork/1d4e7cf3b60af40e008b2e2413f7a2d1ff784b52/kubernetes/icon/color/kubernetes-icon-noborder-color.png'"
            />
            </div>
            <div>
              Kubernetes
            </div>
          </div>
          <div class="environment-divider dash">
          </div>
          <div class="test-env-version">
            <md-select v-model="releaseType" name="releaseType" id="release-type" v-on:selected="selectEnv($event)">
              <md-option value="stable"> 
                Stable {{ ReleaseTag(this.$props.project, 'stable') }}
              </md-option>
              <md-option value="head"> 
                Head {{ ReleaseTag(this.$props.project, 'head') }}
              </md-option>
            </md-select>
          </div>
          <div class="environment-divider">
              <i class="fa fa-arrow-right"></i>
          </div>

          <div class="stage">Bare Metal (Packet)</div>

          <div class="environment-divider">
              <i class="fa fa-arrow-right"></i>
          </div>
          <StatusLabel :url="ReleaseURL(this.$props.project, releaseType)" 
          :label="ReleaseStatus(this.$props.project, releaseType)"
          :branch="'stable'"
         :class="ReleaseStatus(this.$props.project, releaseType)"/>
      </div>

      <md-dialog md-open-from="#custom" md-close-to="#custom" ref="dialog1">
        <md-dialog-title>Select K8s Environment</md-dialog-title>
        <md-dialog-content>
          <md-list>
              <md-list-item @click="closeDialog('dialog1', 'stable')">
                <span>
                  Stable {{ ReleaseTag(this.$props.project, 'stable') }}
                </span>
                <md-icon>{{ListIcon('stable', releaseType)}}</md-icon>
              </md-list-item>

              <md-list-item @click="closeDialog('dialog1', 'head')">
                <span>
                  Head {{ ReleaseTag(this.$props.project, 'head') }}
                </span>
                <md-icon>{{ListIcon('head', releaseType)}}</md-icon>
                </md-list-item>
          </md-list>
        </md-dialog-content>

        <md-dialog-actions>
        <md-button class="md-primary" @click="closeDialog('dialog1')">Cancel</md-button>
        </md-dialog-actions>
      </md-dialog>
    </div>
</template>


<script>
  import Vue from 'vue'
  // import {mapState} from 'vuex'
  import StatusBadge from './StatusBadge'
  import StatusLabel from './StatusLabel'
  import VueMaterial from 'vue-material'
  import 'vue-material/dist/vue-material.css'

  Vue.use(VueMaterial)

  export default {
    name: 'test-environment',
    components: { StatusLabel, StatusBadge },
    data: () => ({
      releaseType: 'stable'
    }),
    props: {
      url: { type: String, default: '' },
      project: { type: Object }
    },
    methods: {
      openDialog (ref) {
        this.$refs[ref].open()
      },
      closeDialog (ref, releaseType) {
        if (releaseType) {
          this.$data.releaseType = releaseType
        }
        this.$refs[ref].close()
      },
      gotoURL () {
        window.open(this.$props.url, '_blank')
      },
      gotoProjectURL () {
        window.open(this.$props.project.url, '_blank')
      },
      selectEnv: function (releaseType) {
        let env = releaseType
        this.$store.dispatch('switchEnv', { env })
      },
      SelectItems (items, releaseType) {
        var selectItems = [items[0]]
        items[0].release_type === releaseType ? selectItems.push(items[1]) : selectItems.unshift(items[1])
        return selectItems
      },
      ListIcon: function (type, releaseType) {
        return type === releaseType ? 'checked' : ''
      },
      ReleaseType: function (type) {
        return type[0].toUpperCase() + type.substring(1)
      },
      ReleaseStatus: function (arg, releaseType) {
        var status = 'N/A'
        if (arg) {
          arg.pipelines.forEach(function (pl) {
            if (pl.release_type === releaseType) {
              // console.log('Stable pipeline' + pl)
              pl.jobs.forEach(function (j) {
                if (j.order === 1) { status = j.status }
              })
            }
          })
        }
        console.log('stable status:' + status)
        return status
      },
      ReleaseURL: function (arg, releaseType) {
        var url = '#'
        if (arg) {
          arg.pipelines.forEach(function (pl) {
            if (pl.release_type === releaseType) {
              pl.jobs.forEach(function (j) {
                if (j.order === 1) {
                  if (!(j.url === null)) {
                    url = j.url
                  }
                  if (j.url === null) {
                    url = '#'
                  }
                }
              })
            }
          })
        }
        return url
      },
      ReleaseTag: function (arg, releaseType) {
        let tag = '#'
        if (arg) {
          arg.pipelines.forEach(function (pl) {
            if (pl.release_type === releaseType) {
              tag = releaseType === 'head' ? pl.head_commit.substring(0, 7) : releaseType === 'stable' ? pl.ref : '#'
              console.log('releaseType' + releaseType)
            }
          })
        }
        return tag
      }
    },
    computed: {
    }
  }

</script>

<style lang="scss">
  @import "../assets/stylesheets/colors";
  @import "../assets/stylesheets/mixins";

  $paddingLeft: calc(#{rem(35)} + #{rem(10)});

  #dashboard-header {
    @include flex-container;
    padding: rem(40) 0;

    @include mq('sm'){ margin-top: 0; }

    #dashboard-logo,
    #dashboard-title { @include fbox(1); }

    #dashboard-logo {

      .cncf-logo {
        width: rem(260);
        cursor: pointer;
        img { width: inherit; }
      }

      @include mq('sm'){
        .cncf-logo { width: rem(180) }
      }
    }

    #dashboard-title {
      text-align: right;
      font-size: rem(24);

     @include mq('sm') { display: none; }

      span.title { text-transform: uppercase; font-weight: bold; }
    }

    @include mq('sm') {
      flex-wrap:wrap;
      padding: rem(20) 0;

      #dashboard-title {
        text-align: left;
        margin: rem(20) 0 0 0;
        font-size: rem(18);
      }
    }
  }

  #dashboard-updated {
    @include mq('md') {
      padding-bottom: 20px;
    }
    .updated-label,
    .time-updated,
    .icon { display: inline-block; }
    .updated-label { font-weight: bold; }
  }

  #test-environment, #test-environment-full {
     align-content: space-around;
     align-items: center;
     background: #FDFDFD;
     border: solid #E5E5E5;
     border-width: 0 1px 1px 1px;
     flex-direction: column;
     justify-content: space-around;
     margin: 0 15px 20px 15px;
     position: relative;
     text-align: center;
     padding: 20px;

      @include mq('md') {
          flex-direction: row;
      }

     &::after, &::before {
      background-color: #E5E5E5;
      content: '';
      width: 30%;
      position: absolute;
      top: -1px;
      height: 1px;
    }
    &::after {
      right: 0;
      @include mq('md') {
       width: calc(100% -  #{rem(25)}  - 140px);
      }
    }
    &::before {
      left: 0;

      @include mq('md') {
        width: calc(#{rem(35)} + #{rem(10)});
      }
    }

    .test-env-label {
         position: absolute;
         top: -8px;
         left: calc(50% - 60px);
         font-weight: 600;
         font-size: rem(14);
         @include mq('md') {
           left: calc(20px);
         }
     }
     .test-env-name, .test-env-details {
         display: flex;
         justify-content: center;
     }
     .test-env-name {
         align-items: center;
         font-weight: 700;
         @include mq('sm') {
             padding-bottom: 20px;
         }
     }
     .test-env-details {
         font-size: rem(14);
         align-items: center;
     }
  }

  #test-environment {
    display: flex;

    @include mq('md') {
      display: none;
    }

    .icon {
      width: rem(35);
      position: relative;

      @include mq('sm') {
        width: 1rem;
        margin-right: 5px;
      }

      img { width: inherit; }
    }

    .test-env-name {
        cursor: pointer;
    }
  }

  #test-environment-full {
    margin: auto;
    box-sizing: border-box;
    padding: 20px rem(35);
    display: none;
    > div {
      flex: 1;
    }

    .test-env-label {
      //left: 20px;
      left: $paddingLeft;
    }

    .test-env-name {
      justify-content: initial;
      padding: 0 rem(10);
      cursor: pointer;
    }
    .environment-divider {
      &.dash {
        position: relative;
        width: 100px;
        flex: initial;
        &::before {
          position: absolute;
          content: '';
          border-top: solid 2px #707070;
          width: 80px;
          top: 50%;
          left: 10px;
        }
      }

      .fa.fa-arrow-right {
        color: #707070;
        font-weight: 400;
      }
    }
 
    .icon {
       width: rem(35);
       height: rem(35);
       padding-right: 10px;
       img {
         width: inherit;
       }
    }
    @include mq('md') {
      display: flex;
    }
  }
  .container {
    @include mq('sm') {
      @include flex-container;
      align-items: baseline;
      align-content: stretch;
    }
    #dashboard-header,
    #dashboard-updated {
      @include mq('sm') {
        @include fbox(1);
        font-size: rem(14);
      }
    }

    #dashboard-updated {
      @include mq('sm') {
        text-align: right;
        position: relative;
        top: rem(-15);

        .icon { display: none; }
        .updated-label,
        .time-updated {
          display: block;
          text-align: right;
        }
      }
    }
  }
  .container {
    justify-content: center;
    padding-top: 20px;
  }
  #test-environment {
    @include mq('sm') {
      background: #FDFDFD;
      margin-bottom: 0;
    }
  }
  .md-theme-default.md-select-content .md-menu-item.md-selected {
    > div > span {
      color: $black;
      font-weight: 700;
    }
  }
  .md-dialog-title.md-title {
    font-weight: 500;
  }
  .md-dialog-content {
    padding: 0;
  }
  .md-select-content {
    display: none;
    @include mq('md') {
      display: block;
      min-width: 150px;
      width: calc((100% / 7.3) + 50px);
    }
 }

   li.md-list-item {
     box-sizing: border-box;
     &.md-menu-item.md-option:first-of-type {
       position: relative;
       &::after {
        color: rgba(0, 0, 0, .38);
        margin-top: 2px;
        position: absolute;
        top: 50%;
        right:  5%;
        transform: translateY(-50%) scaleY(0.45) scaleX(0.85);
        transition: all 0.15s linear;
        content: "\25BC";
        @include mq('md') {
          right: 2%;
        }
        @include mq('lg') {
          right: 10px;
        }
      }
    }
  }

</style>
