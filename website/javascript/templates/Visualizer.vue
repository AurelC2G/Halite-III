<template>
<div class="visuallizer-container">
  <div class="visualizer-row">
    <div class="visualizer-column">
      <div class="game-heading">
        <p class="game-heading-date" v-if="game">{{game.time_played | moment("MMM Do, YY - HH:mm:ss")}}</p>
        <div class="game-heading-players">
          <div class="short">
            <span :class="`player color-${sortedPlayers[0].index + 1}`" v-if="sortedPlayers.length >= 1">
                <a v-if="sortedPlayers[0].user_id" class="player-name-anchor" :href="`/user/?user_id=${sortedPlayers[0].user_id}`">{{sortedPlayers[0].name}}</a>
                <span v-if="!sortedPlayers[0].user_id" class="player-name-anchor">{{sortedPlayers[0].name}}</span>
            </span>
            <span class="action">defeats</span>
            <span :class="`player color-${sortedPlayers[1].index + 1}`" v-if="sortedPlayers.length >= 2">
                <a class="player-name-anchor" :href="`/user/?user_id=${sortedPlayers[1].user_id}`">{{sortedPlayers[1].name}}</a>
              </span>
            <span class="action" v-if="sortedPlayers.length > 2">+{{sortedPlayers.length - 2}}</span>
          </div>
          <div class="long">
            <span :class="`player color-${sortedPlayers[0].index + 1}`" v-if="sortedPlayers.length >= 1">
                <a v-if="sortedPlayers[0].user_id" class="player-name-anchor" :href="`/user/?user_id=${sortedPlayers[0].user_id}`">{{sortedPlayers[0].name}}</a>
                <span v-if="!sortedPlayers[0].user_id" class="player-name-anchor">{{sortedPlayers[0].name}}</span>
            </span>
            <span class="action">defeats</span>
            <span :class="`player color-${player.index + 1}`" v-for="(player, index) in sortedPlayers" v-if="index > 0" :key="index">
                <a v-if="player.user_id" class="player-name-anchor" :href="`/user/?user_id=${player.user_id}`">{{player.name}}</a>
                <span v-if="!player.user_id" class="player-name-anchor" :href="`/user/?user_id=${player.user_id}`">{{player.name}}</span>
            </span>
          </div>
        </div>
      </div>
      <div class="game-replay">
        <div :class="{ 'game-replay-viewer': true, 'recording': recording }"></div>
        <div class="game-replay-controller">
          <div class="game-replay-btn-table">
            <div class="game-replay-btn-cell">
              <span class="replay-btn">
                  <a href="javascript:;" @click="toggleSpeed"><span v-html="speedLabel"></span></a>
              </span>
              <span class="replay-btn">
                  <a href="javascript:;" @click="prevFrame"><span class="icon-prev"></span></a>
              </span>
              <span v-if="!playing" class="replay-btn" style="text-align: center">
                  <a href="javascript:;" @click="playVideo"><span class="icon-play"></span></a>
              </span>
              <span v-if="playing" class="replay-btn" style="text-align: center">
                  <a href="javascript:;" @click="pauseVideo"><span class="icon-pause"></span></a>
              </span>
              <span class="replay-btn">
                  <a href="javascript:;" @click="nextFrame"><span class="icon-next"></span></a>
              </span>
              <span class="replay-btn reset-btn" style="text-align: center">
                  <a href="javascript:;" @click="resetView" title="Reset zoom/pan"><span class="fa fa-refresh"></span></a>
              </span>
              <span class="replay-btn" style="text-align: center">
                     <a href="javascript:;" @click="recordView" title="Record video of game"><span class="fa fa-video-camera"></span></a>
              </span>
              <span class="replay-btn">
                  <a style="text-align: center; margin-bottom: 4px;" v-if="game && game.game_id" :href="replay_download_link(game.game_id)">
                    <span class="icon-download"></span>
              </a>
              </span>
            </div>
            <div class="game-replay-progress">
              <div class="game-replay-progress-inner">
                <div>0</div>
                <div class="game-replay-progress-bar">
                  <vue-slider v-model="frame" ref="slider" v-bind="sliderOptions" @callback="changeFrame"></vue-slider>
                </div>
                <div>{{sliderOptions.max}}</div>
              </div>
            </div>
          </div>
        </div>
      </div>

    </div>
    <div class="visualizer-stats-column">
      <div class="statistics stats-panel">
        <label class="panel-name">GAME/MAP STATS</label>
        <ul class="panel-body list-hori">
          <li class="list-item" :title="`Halite v${replay.ENGINE_VERSION}`">
            Map
            <br>
            Size:
            <span style="font-size: 1em">{{`${replay.production_map.width}x${replay.production_map.height}`}}</span>
            <br>
            Seed:
            <span style="font-size: 1em">{{replay.map_generator_seed}}</span>
          </li>
          <li class="list-item">
            Halite Available
            <br>
            <span>{{stats && stats.frames[frame].remainingHalite}}</span>
          </li>
          <li class="list-item">
            Halite Collected
            <br>
            <span>{{stats && ((1 - (stats.frames[frame].remainingHalite/stats.totalHalite)) * 100).toFixed(2)}}%</span>
          </li>
        </ul>
      </div>
      <div class="stats-panel plyer">
        <label class="panel-name">PLAYER STATS</label>
        <div class="panel-body not-padding">
           <PlayerDetail :replay="replay" :statistics="statistics" :stats="stats" :frame="frame" :chartData="chartData.energy"></PlayerDetail>
        </div>
      </div>
      <div class="stats-panel map-object">
        <label class="panel-name">MAP OBJECT PROPERTIES</label>
        <div class="panel-body">
          <div v-if="selectedPlanet" class="map-object">
            <SelectedPlanet :selected-planet="selectedPlanet" :players="players"></SelectedPlanet>
          </div>
          <div v-if="selectedPoint && !selectedPlanet" class="map-object">
            <SelectedPoint :selected-point="selectedPoint" :players="players"></SelectedPoint>
          </div>
          <div v-if="selectedShip" class="map-object">
            <SelectedShip :selected-ship="selectedShip" :players="players"></SelectedShip>
          </div>
          <div class="message-box" v-if="!selectedPoint && !selectedShip">
            <p><span class="icon-info"></span></p>
            <p>Click on a ship, dropoff, or other map location to see properties</p>
          </div>
        </div>
      </div>
      <div class="stats-panel">
        <label class="panel-name">THEME</label>
        <div>
          <v-select :options="themes" @input="changeTheme" :value="selectedTheme" style="display: inline-block;width:250px" id="theme">
          </v-select>
        </div>
      </div>
    </div>
  </div>
</div>
</template>

<script>
import { saveAs } from 'file-saver/FileSaver';
import Vue from 'vue'
import vSelect from 'vue-select'
import * as api from '../api'
import * as utils from '../utils'
import moment from 'vue-moment'
import vueSlider from 'vue-slider-component'
import VisualizerPanel from './VisualizerPanel.vue'
import PlayerDetail from './PlayerDetail.vue'
import SelectedPlanet from './SelectedPlanet.vue'
import SelectedShip from './SelectedShip.vue'
import SelectedPoint from './SelectedPoint.vue'
import TierPopover from './TierPopover.vue'
import {
  tierClass
} from '../utils'
import _ from 'lodash'

const speedList = {
  2: '&frac12x',
  4: '1x',
  8: '2x',
  12: '3x',
  16: '4x',
  20: '5x',
}

const loadGame = (game) => {
  return import ( /* webpackChunkName: "libhaliteviz" */ "libhaliteviz")
    .then((libhaliteviz) => {
      // just for electron
      return libhaliteviz
        .setAssetRoot((window && window.process && window.process.type) ?
          'assets/js/' : '')
        .then(() => libhaliteviz.parseReplay(game.replay))
    });
};

export default {
  name: 'haliteTV',
  props: {
    game: Object,
    replay: Object,
    width: {
      default: 650,
      required: false,
      type: Number
    },
    height: {
      default: 650,
      required: false,
      type: Number
    },
    dashboard: {
      required: false,
      default: true,
      type: Boolean,
    },
    autoplay: {
      required: false,
      default: true,
      type: Boolean,
    },
  },
  data: function () {
    return {
      themes: [],
      selectedTheme: null,
      baseUrl: '',
      frame: 0,
      time: 0,
      playing: false,
      speedIndex: 3,
      speedLabel: '3x',
      stats: null,
      // sharePopup: false, // issues #361: Remove Share button & Share capabilities
      isHoliday: true,
      showHoliday: false,
      showChartPanel: true,
      // isMobile: window.mobileAndTabletcheck(), // issues #361
      user: null,
      showChart: false,
      recording: false,
      selected: {
        kind: '',
        id: 0,
        owner: '',
        x: 0,
        y: 0,
      },
      sliderOptions: {
        min: 0,
        max: 0,
        speed: 0.5,
        sliderStyle: {
          backgroundColor: '#E6AB00',
          top: 0,
          width: '6px',
          height: '6px',
          left: '4px'
        },
        processStyle: {
          backgroundColor: '#E6AB00'
        },
        tooltipStyle: {
          backgroundColor: '#E6AB00',
          border: '1px solid #E6AB00',
          color: '#30242F'
        },
        piecewiseStyle: {
          backgroundColor: '#23242b'
        }
      },
      players: [],
      sortedPlayers: [],
      selectedPlayers: [],
      zoom: 1,
      pan: {
        x: 0,
        y: 0
      },
    }
  },
  components: {
    vSelect,
    vueSlider,
    PlayerDetail,
    VisualizerPanel,
    SelectedPlanet,
    SelectedShip,
    SelectedPoint,
    TierPopover
  },
  mounted: function () {
    const width = document.body.offsetWidth;
    if(width < 768) {
      const canvasWidth = width - 30;
      this.width = canvasWidth;
      this.height = canvasWidth;
    }

    // Grab a bit more vertical space
    document.querySelector('.navbar-fixed-top').style.position = 'absolute'
    this.getSortedPlayers()
    this.sliderOptions = Object.assign(this.sliderOptions, {
      max: this.replay.full_frames.length - 1,
      value: this.frame
    })

    this.showHoliday = false;

    // current user
    api.me().then((user) => {
      this.user = user;
    });

    if (window.localStorage['holiday'] === undefined || window.localStorage['holiday'] === 'true') {
      this.isHoliday = true;
    } else {
      this.isHoliday = false;
    }

    // Make sure canvas fits on screen
    const onResize = () => {
      if (document.querySelector("canvas")) {
        const windowHeight = window.innerHeight
        const canvasHeight = this.height
        const ratio = canvasHeight / windowHeight
        const factor = 1 / (ratio + 0.2)

        if (ratio > 0.9) {
          document.querySelector("canvas").style.width = `${factor * this.width}px`
          document.querySelector("canvas").style.height = `${factor * this.height}px`
        } else {
          document.querySelector("canvas").style.width = `${this.width}px`
          document.querySelector("canvas").style.height = `${this.height}px`
        }
      }
    }
    window.addEventListener("resize", onResize)

    import ( /* webpackChunkName: "libhaliteviz" */ "libhaliteviz")
    .then((libhaliteviz) => {
      this.themes = Object.keys(libhaliteviz.theme.THEMES);
      this.selectedTheme = libhaliteviz.theme.selectedTheme;
      const visualizer = new libhaliteviz.HaliteVisualizer(this.replay, this.width, this.height)
      this.getVisualizer = function () {
        return visualizer
      }
      const storedSpeedIndex = sessionStorage.getItem('halite-replaySpeed')
      if (storedSpeedIndex) {
        const speedIndex = parseInt(storedSpeedIndex)
        this.speedIndex = speedIndex
        const value = Object.keys(speedList)[speedIndex]
        const label = speedList[value]
        this.speedLabel = label
        visualizer.playSpeed = value
      } else {
        visualizer.playSpeed = 6
      }
      this.stats = visualizer.stats

      visualizer.onUpdate.add(() => {
        this.frame = visualizer.frame
        this.time = visualizer.time
        this.zoom = visualizer.camera.scale / visualizer.camera.initScale
        const camera = visualizer.camera
        this.pan.x = (camera.cols - camera.pan.x) % camera.cols
        this.pan.y = (camera.rows - camera.pan.y) % camera.rows
      })
      visualizer.onPlay.add(() => {
        this.playing = true
      })
      visualizer.onPause.add(() => {
        this.playing = false
      })
      visualizer.onSelect.add((kind, args) => {
        this.selected.kind = kind
        this.selected.id = args.id
        this.selected.owner = args.owner
        this.selected.x = args.x
        this.selected.y = args.y
        this.selected.production = args.production
        this.$refs.objectPanel.open()
        visualizer.onUpdate.dispatch()
        this.$forceUpdate()
        this.gaData('visualizer', 'click-map-objects', 'gameplay')
      })
      visualizer.attach('.game-replay-viewer')
      onResize()
      // play the replay - delay a bit to make sure assets load/are rendered
      if (this.autoplay) {
        window.setTimeout(function () {
          visualizer.play()
        }, 500);
      }

      // action
      this.playVideo = (e) => {
        if (visualizer) {
          if (this.frame >= this.replay.game_statistics.number_turns - 1) {
            visualizer.scrub(0, 0)
            this.frame = 0
            this.time = 0.0
          }
          visualizer.play()
          this.gaData('visualizer', 'click-play', 'gameplay')
        }
      }
      this.pauseVideo = (e) => {
        if (visualizer) {
          visualizer.pause()
        }

        this.gaData('visualizer', 'click-pause', 'gameplay')
      }
      this.resetView = () => {
        if (visualizer) {
          visualizer.camera.reset();
        }
      }
      this.recordView = () => {
        if (visualizer) {
          this.recording = true;
          visualizer.encodeVideo().then((blob) => {
            this.recording = false;
            if (this.game && this.game.game_id) {
              saveAs(blob, `${this.game.game_id}.webm`);
            }
            else {
              saveAs(blob, 'video.webm');
            }
          });
        }
      }
      this.snapshot = () => {
        window.prompt("Copy the snapshot:", visualizer.snapshot())
      }

      const changeSpeed = (speed) => {
        this.speedIndex = speed
        if (this.speedIndex >= Object.keys(speedList).length) this.speedIndex = 0

        const value = Object.keys(speedList)[this.speedIndex]
        const label = speedList[value]
        this.speedLabel = label

        if (visualizer) {
          visualizer.playSpeed = value
        }

        this.gaData('visualizer', 'click-speed', 'gameplay')

        sessionStorage.setItem('halite-replaySpeed', this.speedIndex)
      }

      this.toggleSpeed = (e) => {
        changeSpeed(this.speedIndex + 1);
      }

      this.prevFrame = () => {
        if (visualizer && this.frame > 0) {
          visualizer.scrub(this.frame + -1, 0)
        }

        this.gaData('visualizer', 'click-back', 'gameplay')
      }
      this.nextFrame = () => {
        if (visualizer && this.frame < this.replay.full_frames.length - 1) {
          visualizer.scrub(this.frame + 1, 0)
        }

        this.gaData('visualizer', 'click-forward', 'gameplay')
      }
      this.changeFrame = (event) => {
        // waiting for the slider dot finish to move
        setTimeout(() => {
          if (visualizer) {
            visualizer.scrub(this.frame, 0)
          }
        }, 200)

        this.gaData('visualizer', 'click-slider', 'gameplay')
      }

      this.toggleHoliday = function () {
        if (window.localStorage['holiday'] === undefined || window.localStorage['holiday'] === 'true') {
          window.localStorage['holiday'] = "false";
          this.isHoliday = false;
        } else {
          window.localStorage['holiday'] = "true";
          this.isHoliday = true;
        }
      }

      setTimeout(() => {
        this.$refs.slider.refresh();
      }, 2000);
    })
  },
  computed: {
    statistics: function () {
      const count = {}
      if (!this.entitiesProduced) {
        this.entitiesProduced = []
        for (let frame of this.replay.full_frames) {
          let thisFrame = {
            0: 0,
            1: 0,
            2: 0,
            3: 0
          }
          if (this.entitiesProduced.length > 0) {
            thisFrame[0] = this.entitiesProduced[this.entitiesProduced.length - 1][0]
            thisFrame[1] = this.entitiesProduced[this.entitiesProduced.length - 1][1]
            thisFrame[2] = this.entitiesProduced[this.entitiesProduced.length - 1][2]
            thisFrame[3] = this.entitiesProduced[this.entitiesProduced.length - 1][3]
          }
          if (frame.events) {
            for (let event of frame.events) {
              if (event.event === 'spawn') {
                thisFrame[event.owner_id]++
              }
            }
          }
          this.entitiesProduced.push(thisFrame)
        }
      }

      for (let i = 0; i < this.replay.number_of_players; i++) {
        count[i] = {
          ships: Object.keys(this.replay.full_frames[this.frame].entities[i] || {}).length,
          dropoffs: 0,
          collisions: 0,
          shipsProduced: this.entitiesProduced[this.frame][i],
        }
      }

      return count
    },
    chartData: function () {
      let output = {
        energy: [],
        fleet: []
      }

      if (!this.stats || !this.stats.frames || !this.stats.frames.length || !this.stats.frames[0].players) return output
      for (let _pIndex in this.stats.frames[0].players) {
        let playerEnergy = []
        let playerFleet = []
        this.stats.frames.forEach((_frame, _fIndex) => {
          playerEnergy.push({
            x: _fIndex,
            y: _frame.players[_pIndex].currentEnergy
          })
          playerFleet.push({
            x: _fIndex,
            y: _frame.players[_pIndex].currentShips
          })
        })
        output.energy.push(playerEnergy)
        output.fleet.push(playerFleet)
      }
      return output
    },
    selectedPlanet: function () {
      if (this.selected.kind) {
        let x
        let y
        if (this.selectedShip) {
          ({
            x,
            y
          } = this.selectedShip)
        } else {
          ({
            x,
            y
          } = this.selectedPoint)
        }

        for (const player of this.replay.players) {
          if (player.factory_location.x === x &&
            player.factory_location.y === y) {
            return {
              x,
              y,
              owner: player.player_id,
              type: "Shipyard",
            }
          }
        }

        for (const dropoff of this.getVisualizer().dropoffs) {
          if (dropoff.factoryBase.x === x &&
              dropoff.factoryBase.y === y) {
            return {
              x, y,
              owner: dropoff.owner,
              type: "Dropoff",
            }
          }
        }
      }
    },
    selectedShip: function () {
      if (this.selected.kind === "ship") {
        const state = this.getVisualizer().findShip(this.frame, this.selected.owner, this.selected.id);
        if (state) {
          return Object.assign({}, state, {
            owner: this.selected.owner,
            id: this.selected.id,
          });
        }
      }
    },
    selectedPoint: function () {
      if (this.selected.kind === 'point' ||
        this.selected.kind === 'ship') {
        if (this.selected.kind === 'ship' && this.selectedShip) {
          this.selected.x = this.selectedShip.x;
          this.selected.y = this.selectedShip.y;
        }
        const energy = this.getVisualizer().findCurrentProduction(this.frame, this.selected.x, this.selected.y);
        return {
          energy,
          x: this.selected.x,
          y: this.selected.y,
        };
      }
      return null
    },
    shareLink: function () {
      // const game_id = this.game.game_id;
      // const replay_class = this.game.game.replay_class;
      // const replay = this.game.game.replay;
      return window.location.href
      // return window.location `?game_id=${game_id}&replay_class=${replay_class}&replay_name=${encodeURIComponent(replay)}`
    },
  },
  methods: {
    changeTheme(e) {
      if (e === this.selectedTheme) {
        return;
      }
      import ( /* webpackChunkName: "libhaliteviz" */ "libhaliteviz")
        .then((libhaliteviz) => {
          libhaliteviz.theme.setTheme(e);
          window.location.reload();
        });
    },
    userlink: function (user_id) {
      if (user_id) {
        return `/user/?user_id=${user_id}`
      } else {
        return ''
      }
    },
    tierClass: tierClass,
    getPlayers: async function () {
      if (!this.replay) return []
      return this.replay.game_statistics.player_statistics
        .map((p) => {
          const player = Object.assign({}, p);
          player.name = this.replay.players[player.player_id].name;
          player.index = player.player_id;

          if (this.game) {
            for (const [userId, user] of Object.entries(this.game.players)) {
              if (player.player_id === user.player_index) {
                player.user_id = userId;
                break;
              }
            }
          }

          return player;
        })
    },
    getSortedPlayers: async function () {
      const players = await this.getPlayers()
      this.players = _.sortBy(players, ['player_id'])
      this.sortedPlayers = _.sortBy(players, ['rank'])

      const selectedPlayers = []
      this.players.forEach(function (item, index) {
        selectedPlayers[index] = true
      })
      this.selectedPlayers = selectedPlayers
    },
    getPlayerName: function (botname) {
      return botname.replace(/\sv\d+$/, '')
    },
    playVideo: function (event) {},
    pauseVideo: function (event) {},
    toggleSpeed: function (event) {},
    prevFrame: function () {},
    nextFrame: function () {},
    changeFrame: function (event) {},
    resetView: function () {},
    recordView: function () {},
    snapshot: function () {},
    toggleChartPanel: function (e) {
      this.showChartPanel = !this.showChartPanel
      sessionStorage.setItem('halite-showChartPanel', this.showChartPanel.toString())
      this.initChart()
    },
    initChart: function () {
      if (this.showChart) return
      setTimeout(() => this.showChart = true, 500)
    },
    gaData: function (category, action, label) {
      utils.gaEvent(category, action, label)
    },
    // issues #361: Remove Share button & Share capabilities
    // toggleShare: function () {
    //   this.sharePopup = !this.sharePopup
    // },
    shareSocial: function (social) {
      let text = 'Halite Game Replay - '
      let tags = 'halitegame'
      switch (social) {
        case 'facebook':
          return 'https://www.facebook.com/sharer.php?u=' + encodeURIComponent(window.location.href)
          break
        case 'twitter':
          return 'https://twitter.com/intent/tweet?text=' + text + '&url=' + encodeURIComponent(window.location.href) + '&hashtags=' + tags + '&via=haliteAI'
          break
        case 'linkedin':
          return `https://www.linkedin.com/shareArticle?mini=true&url=${encodeURIComponent(window.location.href)}`
          break
      }
    },
    /**
     * @param  {e} event
     * @return {void}
     */
    copyToClipboard: function (e) {
      if (e) e.preventDefault()
      this.$refs.shareInput.select()
      document.execCommand('copy')
    },
    /**
     * Download link
     */
    replay_download_link: function (game_id) {
      let user_id = this.user ? this.user.user_id : 0
      return `${api.API_SERVER_URL}/user/${user_id}/match/${game_id}/replay`
    },
  }
}
</script>

<style lang="scss">
.game-replay-viewer {
  position: relative;
  canvas {
    display: block;
    margin: 0 auto;
  }
}
.game-replay-viewer.recording::after {
  display: block;
  content: "Recording…";
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  background: rgba(0, 0, 0, 0.5);
  font-size: 4em;
  animation: blink 1s infinite alternate;
}
@keyframes blink {
  0% {
    color: rgba(151, 221, 255, 0);
  }
  100% {
    color: rgba(151, 221, 255, 1);
  }
}

.stats-panel{
  label{
    font-weight: normal;
  }
  .panel-name{
    margin-bottom: 0;
    color: #ECFFFB;
    font-size: 16px;
    letter-spacing: 0.6px;
    line-height: 19px;
  }
  .panel-body{
    display: flex;
    margin: 16px 0;
    background: #fff;
    border-radius: 4px;
   //@at-root box-shadow: inset 0 1px 3px 0 rgba(0,0,0,0.5), inset 0 1px 14px 0 rgba(151,182,255,0.45);
    &.not-padding {
      padding: 0;
      overflow: hidden;
    }

    >div {
      flex: 1;

      &:last-child {
        margin-right: 0;
      }

      &.map-object {
        max-width: calc(49% - 1em);
        margin-right: 1em;
      }
    }
  }
  .list-hori{
    list-style: none;
  }
  .list-hori{
    padding: 0;
    .list-item{
      display: inline-block;
      width: 32.9%;
      border-right: 1.2px solid rgba(8,27,83, .1);
      padding: 10px 15px;
      font-size: 14px;
      line-height: 17px;
      color: #1B1B1B;
      &:last-child {
      border-right: none;
      }
      span{
        color: #003285;
        font-size: 18px;
        font-weight: 500;
        letter-spacing: 0.68px;
        line-height: 22px;
        margin-top: 5px;
      }
    }
  }
}

@media (max-width: 768px) {
  .visualizer-column {
    margin: 0 auto !important;
    .game-replay .game-replay-viewer {
      padding: 10px 0;
      canvas {
        margin: 0;
      }
    }
  }
}
</style>
