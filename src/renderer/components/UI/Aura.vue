<template>
  <div
    class="aura"
    v-bind:class="{
      notAnUpdate: showAllAuras && aura.version == aura.wagoVersion
    }"
  >
    <transition name="fade-header">
      <div class="aura-closed" v-if="!toggled" @click="toggleInfo">
        <a
          class="wago_icon"
          target="_blank"
          :href="wagoURL(aura.slug)"
          v-tooltip="{
            content: wagoURL(aura.slug),
            html: false
          }"
        />
        <div class="aura_name_container">
          <span class="aura_name">{{ aura.name }} </span>
        </div>
        <div
          v-if="showAllAuras && aura.version < aura.wagoVersion"
          class="update-ready"
        >
          {{ $t("app.aura.aura-ready" /* Update ready in-game */) }}
        </div>
        <a
          class="author"
          target="_blank"
          :href="wagoAuthorURL(aura.author)"
          v-tooltip="{
            content: wagoAuthorURL(aura.author),
            classes: ['small'],
            html: false
          }"
        >
          {{ aura.author }}
        </a>
        <div class="upgrade-text">
          <div class="current-version">
            v<span v-if="aura.semver">{{ aura.semver }}</span>
            <span v-else>{{ aura.version }}</span>
          </div>
          <div
            class="wago-version"
            v-tooltip="{
              content: fromNow(currentTime, $i18n.locale),
              classes: ['small']
            }"
            @mouseover="updateCurrentTime()"
          >
            v<span v-if="aura.wagoSemver">{{ aura.wagoSemver }}</span>
            <span v-else>{{ aura.wagoVersion }}</span>
          </div>
        </div>
      </div>
    </transition>
    <transition name="fade">
      <div class="aura-open" v-if="toggled">
        <div class="close-aura"></div>
        <div class="aura-header" @click="toggleInfo">
          <span class="name">{{ aura.name }}</span>
          <span class="version"> {{ aura.wagoSemver }}</span>
          <span class="author">{{ aura.author }}</span>
        </div>
        <div class="aura-body">
          <div class="changelog">
            <div class="aura-title">
              {{ $t("app.aura.changelog" /* Changelog */) }}
            </div>
            <div class="content">{{ childs }}</div>
          </div>
          <div class="current-info">
            <div class="aura-title">
              {{ $t("app.aura.currentlyInstalled" /* Currently Installed */) }}
            </div>
            <table>
              <tr>
                <td class="label">
                  {{ $t("app.aura.currentVersion" /* Version */) }}
                </td>
                <td class="value" v-if="aura.semver">v{{ aura.semver }}</td>
                <td class="value" v-else>{{ aura.version }}</td>
              </tr>
              <tr>
                <td class="label">{{ $t("app.aura.link" /* Link */) }}</td>
                <td class="value">
                  <a :href="wagoURL(aura.slug)">{{ wagoURL(aura.slug) }}</a>
                </td>
              </tr>
              <tr v-if="showAllAuras">
                <td class="label">{{ $t("app.aura.status" /* Status */) }}</td>
                <td class="value status">
                  <div
                    v-if="aura.version < aura.wagoVersion"
                    class="update-ready"
                  >
                    {{ $t("app.aura.aura-ready" /* Update ready in-game */) }}
                  </div>
                  <div v-else class="up-to-date">
                    {{ $t("app.aura.uptodate" /* Up to Date */) }}
                  </div>
                </td>
              </tr>
            </table>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import moment from "moment";
import Vue from "vue";
import VTooltip from "v-tooltip";

const sanitize = require("../libs/sanitize.js");

Vue.use(VTooltip);
export default Vue.extend({
  props: ["aura", "showAllAuras"],
  data() {
    return {
      currentTime: null,
      toggled: false
    };
  },
  computed: {
    childs() {
      let output = "";
      if (this.aura.ids) {
        if (this.aura.changelog && this.aura.changelog.text) {
          if (this.aura.changelog.format === "bbcode") {
            output += sanitize.bbcode(this.aura.changelog.text);
          } else if (this.aura.changelog.format === "markdown") {
            output += sanitize.markdown(this.aura.changelog.text);
          }
          output += "\n\n";
        }
        const { ids } = this.aura;
        output += ids.sort().join("\n");
      }
      return output;
    }
  },
  methods: {
    updateCurrentTime() {
      this.currentTime = null;
      this.currentTime = this.aura.modified;
    },
    wagoURL(value) {
      if (!value) return "";
      return `https://wago.io/${value}`;
    },
    wagoAuthorURL(author) {
      if (!author) return "";
      return `https://wago.io/p/${author}`;
    },
    fromNow(value, locale) {
      if (!value) return "n/a";
      return moment(value)
        .locale(locale)
        .fromNow();
    },
    toggleInfo() {
      this.toggled = !this.toggled;
    }
  }
});
</script>

<style scoped lang="scss">
$accent: rgb(255, 209, 0);
.aura {
  .aura-closed {
    background-color: #101010c9;
    border: 1px solid #101010;
    color: $accent;
    margin: 1px;
    height: 36px;
    max-height: 36px;
    padding: 2px 10px;
    vertical-align: middle;
    white-space: nowrap;
    display: flex;
    overflow: hidden;
    font-size: 11px;
    flex-direction: row;
    border-radius: 4px;
    &:hover {
      background-color: #161616c9;
    }
    .aura_name_container {
      text-align: left;
      flex: 1;
      overflow: hidden;
      .aura_name {
        width: 95%;
        overflow: hidden;
        text-overflow: ellipsis;
        padding-left: 10px;
        display: inline-block;
        font-weight: 600;
        font-size: 12px;
        text-shadow: #000000 0 1px 0;
        text-align: left;
        line-height: 32px;
      }
    }
    .update-ready {
      float: right;
      line-height: 32px;
    }
    .wago_icon {
      height: 27px;
      margin: auto;
      vertical-align: bottom;
      display: inline-block;
      border-radius: 8px;
      background: rgba(0, 0, 0, 0.3);
      content: url("~@/assets/wago_plain.png");
    }
    .author {
      width: 100px;
      overflow: hidden;
      text-overflow: ellipsis;
      padding-left: 10px;
      font-size: 12px;
      text-align: left;
      line-height: 32px;
      &:before {
        content: "@";
      }
    }
    .upgrade-text {
      font-weight: 600;
      text-shadow: #000000 0 1px 0;
      display: flex;
    }
    .current-version {
      font-size: 9px;
      color: #777;
      width: 55px;
      line-height: 32px;
      text-align: left;
      padding-top: 1px;
    }
    .wago-version {
      text-shadow: rgba(219, 185, 50, 0.267) 0 0 5px;
      text-align: right;
      width: 55px;
      line-height: 32px;
    }
  }
  .aura-open {
    max-height: 500px;
    background-color: #131313;
    border: 1px solid #101010;
    transition: height ease-in-out 0.4s;
    border-radius: 8px;
    color: #e6e6e6;
    .aura-title {
      margin-bottom: 5px;
      font-weight: 500;
      font-size: 14px;
      width: auto;
      color: $accent;
    }
    .aura-header {
      text-align: left;
      padding-right: 20px;
      padding: 10px 35px 10px 15px;
      border-radius: 8px 8px 0 0;
      height: 40px;
      background-image: url("~@/assets/wago-aura.png");
      background-size: contain;
      background-position-x: 0;
      background-repeat: no-repeat;
      background-color: #101010;
      transition: background-color ease-in-out 0.2s;
      &:hover {
        background-color: #161616c9;
      }
      .name {
        font-weight: 500;
        text-align: left;
        color: $accent;
        text-shadow: #000000 0 0 10px;
      }
      .author,
      .version {
        margin: 0 5px;
        float: right;
      }
      .author {
        order: 1;
        font-size: 10px;
        line-height: 20px;
        color: #777;
        &:before {
          content: "@";
        }
      }
      .version {
        text-shadow: rgba(219, 185, 50, 0.267) 0 0 5px;
        color: $accent;
        font-weight: 500;
        font-size: 14px;
      }
    }
    .aura-body {
      text-align: left;
      padding: 10px 15px 15px;
      .changelog {
        width: 50%;
        display: inline-block;
        .content {
          border-radius: 8px;
          padding: 10px 0;
          font-size: 12px;
          white-space: pre-line;
          text-align: left;
          display: block;
          max-height: 200px;
          width: 100%;
          overflow: auto;
        }
      }
      .current-info {
        width: 49%;
        display: inline-block;
        vertical-align: top;
        text-align: right;
        table {
          text-align: left;
          float: right;
          font-size: 12px;
          border-radius: 8px;
          td {
            padding: 2px 8px 2px 0;
            &:not(:first-child) {
              min-width: 200px;
            }
            &.label {
              font-weight: 600;
            }
            .update-ready {
              &:after {
                content: "";
                width: 20px;
                height: 20px;
                padding: 10px;
                top: 8px;
                margin-left: 2px;
                position: relative;
                background: url("~@/assets/wow-logo.svg");
                background-size: contain;
                background-repeat: no-repeat;
              }
            }
            .up-to-date {
              &:after {
                content: "\e92d";
                font-family: "Material Icons";
                font-size: 22px;
                line-height: 0px;
                color: #51ae42;
                position: relative;
                top: 6px;
              }
            }
          }
        }
      }
    }
    .close-aura {
      width: 100%;
      height: 0px;
      position: relative;
      right: 10px;
      top: 9px;
      text-align: right;
      cursor: pointer;
      &:before {
        content: "\e5cd";
        font-family: "Material Icons";
        font-size: 20px;
        color: #e6e6e6;
      }
    }
  }

  $animType: ease-in-out 0.5s;
  $animFast: ease-in-out 0.25s;
  // .fade-enter-active,
  // .fade-leave-active {
  //   overflow: hidden;
  //   transition: all $animType;
  //   // transition: height $animType, max-height $animType, color ease-in-out 0.7s,
  //   //   text-shadow $animType, margin $animType, padding $animType,
  //   //   border-width $animType;
  // }
  // .fade-fast-enter-active,
  // .fade-fast-leave-active {
  //   overflow: hidden;
  //   transition: all $animFast;
  // }

  .fade-enter-active {
    overflow: hidden;
    transition: all $animType;
  }

  .fade-header-enter-active,
  .fade-header-leave-active,
  .fade-leave-active {
    overflow: hidden;
    transition: all $animFast;
  }

  .fade-header-enter, .fade-header-leave-to, .fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
    max-height: 0;
    color: transparent;
    text-shadow: none;
    margin: 0;
    padding-top: 0;
    padding-bottom: 0;
    border-width: 0;
  }
}
</style>
