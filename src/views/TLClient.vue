<template>
  <v-container fill-height>
    <v-card
      v-if="MenuBool"
      class="TopMenu"
    >
      <v-container class="d-flex align-baseline">
        <v-btn
          elevation="4"
          color="secondary"
          @click="ModalMode = 3; ModalNexus = true"
        >
          Setting
        </v-btn>
        <v-divider
          vertical
          style="margin-left: 5px; margin-right:5px"
        />
        <v-btn
          elevation="4"
          color="secondary"
          @click="ModalMode = 4; ModalNexus = true; ActiveURLStream = '';"
        >
          Sync Chat
        </v-btn>
        <v-btn
          elevation="4"
          color="secondary"
          style="margin-left:5px"
          @click="ModalMode = 5; ModalNexus = true"
        >
          UnSync Chat
        </v-btn>
      </v-container>
    </v-card>
    <div class="d-flex flex-column" style="height:100%; width:100%">
      <v-btn
        block
        elevation="4"
        color="primary"
        small
        @click="MenuBool = !MenuBool"
      >
        Menu
      </v-btn>
      <div class="d-flex align-stretch flex-row" style="height:100%" @click="MenuBool = false">
        <v-card class="ChatOuterContainer grow" height="100%;" width="auto">
          <v-container class="ChatInnerContainer d-flex flex-column">
            <EnhancedEntry
              v-for="(dt, index) in Entries"
              :key="index"
              :time="dt.Time"
              :stext="dt.SText"
              :c-c="dt.CC"
              :o-c="dt.OC"
            />
          </v-container>
          <v-card v-if="ProfileDisplay" class="ProfileListCard d-flex flex-column">
            <span v-for="(prf, index) in Profile" :key="index"><span v-if="index === ProfileIdx">> </span>{{ (index + 1) + '. ' + prf.Name }}</span>
          </v-card>
        </v-card>
        <v-card
          v-if="ActiveChat.length > 0"
          class="ChatPanelContainer"
          height="100%"
          :width="ActiveChat.length < 2 ? '50%' : '70%'"
          :style="ActiveChatGridRow"
          outlined
        >
          <v-card
            v-for="(ChatURL, index) in ActiveChat"
            :key="ChatURL.text"
            class="d-flex flex-column"
            outlined
          >
            <p class="text-center" style="margin-top: 5px">
              {{ ChatURL.text }}
              <v-icon class="float-right" @click="CloseActiveChat(index)">
                {{ mdiCloseCircle }}
              </v-icon>
            </p>
            <iframe
              class="ActiveChatIFrame"
              :src="URLExtender(ChatURL.text)"
              frameborder="0"
              @load="IFrameLoaded($event, ChatURL.text)"
            />
          </v-card>
        </v-card>
      </div>

      <v-container
        @click="MenuBool = false"
        @keydown.up.exact="ProfileUp()"
        @keydown.down.exact="ProfileDown()"
        @keydown.tab.exact.prevent="ProfileDown()"
        @keydown.shift.tab.exact.prevent="ProfileJumpToDefault()"
        @keydown.ctrl.49.exact.prevent="ProfileJump(0)"
        @keydown.ctrl.50.exact.prevent="ProfileJump(1)"
        @keydown.ctrl.51.exact.prevent="ProfileJump(2)"
        @keydown.ctrl.52.exact.prevent="ProfileJump(3)"
        @keydown.ctrl.53.exact.prevent="ProfileJump(4)"
        @keydown.ctrl.54.exact.prevent="ProfileJump(5)"
        @keydown.ctrl.55.exact.prevent="ProfileJump(6)"
        @keydown.ctrl.56.exact.prevent="ProfileJump(7)"
        @keydown.ctrl.57.exact.prevent="ProfileJump(8)"
      >
        <v-row class="align-baseline">
          <v-text-field
            v-model="InputString"
            @keypress.enter="AddEntry()"
          />
          <v-btn style="margin-left:10px" @click="AddEntry()">
            Enter
          </v-btn>
        </v-row>
        <v-row class="align-stretch">
          <v-col cols="2">
            <v-text-field
              v-model="Profile[ProfileIdx].Prefix"
              label="Prefix"
              dense
              rounded
              outlined
            />
          </v-col>
          <v-col cols="2">
            <v-text-field
              v-model="Profile[ProfileIdx].Suffix"
              label="Suffix"
              dense
              rounded
              outlined
            />
          </v-col>
          <v-checkbox
            v-model="Profile[ProfileIdx].UseCC"
            class="shrink"
            label="Font Colour : "
            hide-details
          />
          <v-btn
            class="ColourButton"
            small
            :style="{ background: Profile[ProfileIdx].CC }"
            @click.stop="ColourTemp = Profile[ProfileIdx].CC; ColourPick = 1; ColourDialogue = true;"
          >
            {{ Profile[ProfileIdx].CC }}
          </v-btn>
          <v-checkbox
            v-model="Profile[ProfileIdx].UseOC"
            label="Outline Colour : "
            hide-details
          />
          <v-btn
            class="ColourButton"
            small
            :style="{ background: Profile[ProfileIdx].OC }"
            @click.stop="ColourTemp = Profile[ProfileIdx].OC; ColourPick = 2; ColourDialogue = true;"
          >
            {{ Profile[ProfileIdx].OC }}
          </v-btn>
          <v-col cols="2" style="margin-left:auto">
            <v-text-field
              v-model="LocalPrefix"
              label="Prefix"
              dense
              rounded
              outlined
            />
          </v-col>
        </v-row>
        <v-row>
          <v-btn @click="ModalMode = 1; ModalNexus = true; AddProfileNameString = 'Profile ' + Profile.length;">
            Add Profile
          </v-btn>
          <v-btn @click="ModalMode = 2; ModalNexus = true">
            Remove Profile
          </v-btn>
          <v-btn @click="ShiftProfileUp()">
            Shift Up
          </v-btn>
          <v-btn @click="ShiftProfileDown()">
            Shift Down
          </v-btn>
        </v-row>
      </v-container>
    </div>

    <!---------   COLOUR MODAL --------->
    <v-dialog
      v-model="ColourDialogue"
      max-width="300px"
      @click:outside.prevent="ColourPickerClose();"
    >
      <v-card>
        <v-color-picker v-if="ColourPick === 1" v-model="Profile[ProfileIdx].CC" />
        <v-color-picker v-else-if="ColourPick === 2" v-model="Profile[ProfileIdx].OC" />
        <v-card-title :style="TextStyle" style="font-weight:bold;">
          The quick brown fox jumps over the lazy dog
        </v-card-title>
        <v-card-actions>
          <v-btn @click="ColourPickerClose();">
            Cancel
          </v-btn>

          <v-btn style="margin-left:auto" @click="ColourPickerOK()">
            Ok
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <!--========   COLOUR MODAL =======-->

    <!---------   NEXUS MODAL ---------
      1 Add profile
      2 Remove Profile
      3 Setting
      4 Load Chat
      5 Unload Chat ALL
    -->
    <v-dialog
      v-model="ModalNexus"
      max-width="600px"
      persistent
      @click:outside="ModalNexusOutsideClick();"
    >
      <!---------    ADD PROFILE     --------->
      <v-card v-if="ModalMode === 1">
        <v-container>
          <v-card-title>
            Add New Profile.
          </v-card-title>
          <v-text-field
            v-model="AddProfileNameString"
            label="Profile Name"
            placeholder="Profile Name"
            dense
            rounded
            outlined
          />
          <v-card-actions>
            <v-btn @click="ModalNexus = false">
              Cancel
            </v-btn>

            <v-btn style="margin-left:auto" @click="AddProfile()">
              Ok
            </v-btn>
          </v-card-actions>
        </v-container>
      </v-card>

      <!---------    Remove PROFILE     --------->
      <v-card v-if="ModalMode === 2">
        <v-container>
          <v-card-title>
            Confirm remove profile {{ Profile[ProfileIdx].Name }}.
          </v-card-title>
          <v-card-actions>
            <v-btn @click="ModalNexus = false">
              Cancel
            </v-btn>

            <v-btn style="margin-left:auto" @click="DeleteProfile()">
              Ok
            </v-btn>
          </v-card-actions>
        </v-container>
      </v-card>

      <!---------    SETTING     --------->
      <v-card v-if="ModalMode === 3">
        <v-container>
          <v-card-title>
            Setting
          </v-card-title>
          <v-select
            v-model="TLLang"
            :items="TL_LANGS"
            :item-text="item => item.text + ' (' + item.value + ')'"
            item-value="value"
            label="TL Language"
            return-object
            @change="LocalPrefix = '[' + TLLang.value + '] '"
          />
          <v-text-field v-model="MainStreamLink" label="Main Stream Link" />
          <v-card-title>Collab Links</v-card-title>
          <v-text-field
            v-for="(AuxLink, index) in CollabLinks"
            :key="index"
            v-model="CollabLinks[index]"
            :append-outer-icon="mdiPlusCircle"
            :prepend-icon="mdiMinusCircle"
            style="margin-left: 17px"
            @click:prepend="DeleteAuxLink(index)"
            @click:append-outer="CollabLinks.push('');"
          />
          <v-card-actions class="d-flex flex-row justify-center">
            <v-btn @click="SettingOKClick()">
              Ok
            </v-btn>
          </v-card-actions>
        </v-container>
      </v-card>

      <!-------  LOAD ACTIVE CHAT ------->
      <v-card v-if="ModalMode === 4">
        <v-container>
          <v-card-title>
            Sync chat
          </v-card-title>
          <v-text-field v-model="ActiveURLStream" label="Stream Link" />
          <v-card-actions>
            <v-btn @click="ModalNexus = false">
              Cancel
            </v-btn>

            <v-btn style="margin-left:auto" @click="LoadChat(ActiveURLStream); ModalNexus = false;">
              Ok
            </v-btn>
          </v-card-actions>
        </v-container>
      </v-card>

      <!-------  UNLOAD ALL ACTIVE CHAT ------->
      <v-card v-if="ModalMode === 5">
        <v-container>
          <v-card-title>
            Unload ALL active chat?
          </v-card-title>
          <v-card-actions>
            <v-btn @click="ModalNexus = false">
              Cancel
            </v-btn>

            <v-btn style="margin-left:auto" @click="UnloadAll(); ModalNexus = false;">
              Ok
            </v-btn>
          </v-card-actions>
        </v-container>
      </v-card>
    </v-dialog>
    <!--========   NEXUS MODAL =======-->
  </v-container>
</template>

<script lang="ts">
import EnhancedEntry from "@/components/tlclient/EnhancedEntry.vue";
import { TL_LANGS } from "@/utils/consts";
import { mdiPlusCircle, mdiMinusCircle, mdiCloseCircle } from "@mdi/js";

export default {
    name: "Tlclient",
    metaInfo() {
        return {
            get title() {
                return "TLClient - Holodex";
            },
        };
    },
    components: {
        EnhancedEntry,
    },
    data() {
        return {
            TL_LANGS,
            mdiPlusCircle,
            mdiMinusCircle,
            mdiCloseCircle,
            MenuBool: false,
            FirstLoad: true,
            Entries: [],
            Profile: [{
                Name: "Default",
                Prefix: "",
                Suffix: "",
                UseCC: false,
                CC: "#000000",
                UseOC: false,
                OC: "#000000",
            }],
            ProfileContainer: {},
            ProfileIdx: 0,
            ProfileDisplay: false,
            ProfileDisplayTimer: undefined,
            InputString: "",
            LocalPrefix: `[${TL_LANGS[0].value}] `,
            // ------ COLOUR -------
            ColourPick: 0,
            ColourDialogue: false,
            ColourTemp: "",
            // ------ MODAL --------
            ModalNexus: true,
            ModalMode: 3,
            AddProfileNameString: "",
            // ------ SETTING ------
            TLLang: TL_LANGS[0],
            MainStreamLink: "",
            CollabLinks: [""],
            // ---- ACTIVE CHAT ----
            ActiveChat: [],
            ActiveURLStream: "",
        };
    },
    computed: {
        TextStyle() {
            return {
                "-webkit-text-fill-color": (this.Profile[this.ProfileIdx].CC === "") ? "unset" : this.Profile[this.ProfileIdx].CC,
                "-webkit-text-stroke-color": (this.Profile[this.ProfileIdx].OC === "") ? "unset" : this.Profile[this.ProfileIdx].OC,
                "-webkit-text-stroke-width": (this.Profile[this.ProfileIdx].OC === "") ? "0px" : "1px",
            };
        },
        ActiveChatGridRow() {
            return ({
                "grid-template-rows": this.ActiveChat.length < 4 ? "1fr" : "1fr 1fr",
            });
        },
    },
    methods: {
        IFrameLoaded(event, target: string) {
            for (let i = 0; i < this.ActiveChat.length; i += 1) {
                if (this.ActiveChat[i].text === target) {
                    this.ActiveChat[i].IFrameEle = event.target;
                }
            }
        },
        AddEntry() {
            this.Entries.push({
                Time: Date.now(),
                SText: this.Profile[this.ProfileIdx].Prefix + this.InputString + this.Profile[this.ProfileIdx].Suffix,
                CC: this.Profile[this.ProfileIdx].UseCC ? this.Profile[this.ProfileIdx].CC : "",
                OC: this.Profile[this.ProfileIdx].UseOC ? this.Profile[this.ProfileIdx].OC : "",
            });

            // SEND TO HOLODEX += 1
            this.ActiveChat.forEach((e) => {
                switch (e.text.slice(0, 3)) {
                    case "YT_":
                        e.IFrameEle?.contentWindow?.postMessage({
                            n: "HolodexSync",
                            d: this.LocalPrefix + this.Profile[this.ProfileIdx].Prefix + this.InputString + this.Profile[this.ProfileIdx].Suffix,
                        }, "https://www.youtube.com");
                        break;

                    case "TW_":
                        e.IFrameEle?.contentWindow?.postMessage({
                            n: "HolodexSync",
                            d: this.LocalPrefix + this.Profile[this.ProfileIdx].Prefix + this.InputString + this.Profile[this.ProfileIdx].Suffix,
                        }, "https://www.twitch.tv");
                        break;

                    default:
                        break;
                }
            });

            // SEND TO API
            this.InputString = "";
        },
        DeleteAuxLink(idx: number) {
            if (this.CollabLinks.length !== 1) {
                this.CollabLinks.splice(idx, 1);
            }
        },
        ModalNexusOutsideClick() {
            if (this.ModalMode !== 3) {
                this.ModalNexus = false;
            }
        },
        SettingOKClick() {
            this.ModalNexus = false;
            if (this.FirstLoad) {
                this.LoadChat(this.MainStreamLink);
                this.CollabLinks.forEach((e) => {
                    this.LoadChat(e);
                });
                this.FirstLoad = false;
            }
        },
        // ------------------------ PROFILE CONTROLLER ------------------------
        ShiftProfileUp() {
            if (this.ProfileIdx > 1) {
                this.ProfileContainer = JSON.parse(JSON.stringify(this.Profile[this.ProfileIdx - 1]));
                this.Profile[this.ProfileIdx - 1] = this.Profile[this.ProfileIdx];
                this.Profile[this.ProfileIdx] = this.ProfileContainer;
                this.ProfileIdx -= 1;
                this.ProfileContainer = {};
            }
            this.ShowProfileList();
        },
        ShiftProfileDown() {
            if ((this.ProfileIdx !== 0) && (this.ProfileIdx < this.Profile.length - 1)) {
                this.ProfileContainer = JSON.parse(JSON.stringify(this.Profile[this.ProfileIdx + 1]));
                this.Profile[this.ProfileIdx + 1] = this.Profile[this.ProfileIdx];
                this.Profile[this.ProfileIdx] = this.ProfileContainer;
                this.ProfileIdx += 1;
                this.ProfileContainer = {};
            }
            this.ShowProfileList();
        },
        ProfileUp() {
            if (this.ProfileIdx === 0) {
                this.ProfileIdx = this.Profile.length - 1;
            } else {
                this.ProfileIdx -= 1;
            }
            this.ShowProfileList();
        },
        ProfileDown() {
            if (this.ProfileIdx === this.Profile.length - 1) {
                this.ProfileIdx = 0;
            } else {
                this.ProfileIdx += 1;
            }
            this.ShowProfileList();
        },
        ProfileJump(idx: number) {
            if (idx < this.Profile.length) {
                this.ProfileIdx = idx;
            }
            this.ShowProfileList();
        },
        ProfileJumpToDefault() {
            this.ProfileIdx = 0;
            this.ShowProfileList();
        },
        AddProfile() {
            if (this.AddProfileNameString.trim() === "") {
                this.AddProfileNameString = `Profile ${this.Profile.length}`;
            }
            this.Profile.push({
                Name: this.AddProfileNameString,
                Prefix: "",
                Suffix: "",
                UseCC: false,
                CC: "#000000",
                UseOC: false,
                OC: "#000000",
            });
            this.ProfileIdx = this.Profile.length - 1;
            this.ModalNexus = false;
            this.ShowProfileList();
        },
        DeleteProfile() {
            if (this.ProfileIdx !== 0) {
                this.ProfileIdx -= 1;
                this.Profile.splice(this.ProfileIdx + 1, 1);
            }
            this.ModalNexus = false;
            this.ShowProfileList();
        },
        ShowProfileList() {
            if (!this.ProfileDisplay) {
                this.ProfileDisplay = true;
            }

            if (this.ProfileDisplayTimer) {
                clearInterval(this.ProfileDisplayTimer);
            }

            this.ProfileDisplayTimer = setInterval(() => {
                this.ProfileDisplay = false;
                clearInterval(this.ProfileDisplayTimer);
            }, 3000);
        },
        //= ======================== PROFILE CONTROLLER ========================

        // ---------------------- ACTIVE CHAT CONTROLLER ----------------------
        UnloadAll() {
            this.ActiveChat = [];
        },
        CloseActiveChat(idx: number) {
            this.ActiveChat.splice(idx, 1);
        },
        URLExtender(s: string) {
            switch (s.slice(0, 3)) {
                case "YT_":
                    return `https://www.youtube.com/live_chat?v=${s.slice(3)}&embed_domain=${window.location.hostname}`;

                case "TW_":
                    return `https://www.twitch.tv/embed/${s.slice(3)}/chat?parent=${window.location.hostname}`;

                default:
                    return "";
            }
        },
        LoadChat(s: string) {
            let StreamURL = s;
            if (StreamURL.indexOf("https://www.youtube.com/watch?v=") === 0) {
                StreamURL = StreamURL.replace("https://www.youtube.com/watch?v=", "");
                if (StreamURL.indexOf("&") !== -1) {
                    StreamURL = StreamURL.slice(0, StreamURL.indexOf("&"));
                }
                StreamURL = `YT_${StreamURL}`;
                if (this.ActiveChat.filter((e) => e.text === StreamURL).length === 0) {
                    this.ActiveChat.push({
                        text: StreamURL,
                        IFrameEle: undefined,
                    });
                }
            } else if (StreamURL.indexOf("https://youtu.be/") === 0) {
                StreamURL = StreamURL.replace("https://youtu.be/", "");
                if (StreamURL.indexOf("?") !== -1) {
                    StreamURL = StreamURL.slice(0, StreamURL.indexOf("?"));
                }
                StreamURL = `YT_${StreamURL}`;
                if (this.ActiveChat.filter((e) => e.text === StreamURL).length === 0) {
                    this.ActiveChat.push({
                        text: StreamURL,
                        IFrameEle: undefined,
                    });
                }
            } else if (StreamURL.indexOf("https://www.youtube.com/live_chat?is_popout=1&v=") === 0) {
                StreamURL = StreamURL.replace("https://www.youtube.com/live_chat?is_popout=1&v=", "");
                if (StreamURL.indexOf("&") !== -1) {
                    StreamURL = StreamURL.slice(0, StreamURL.indexOf("&"));
                }
                StreamURL = `YT_${StreamURL}`;
                if (this.ActiveChat.filter((e) => e.text === StreamURL).length === 0) {
                    this.ActiveChat.push({
                        text: StreamURL,
                        IFrameEle: undefined,
                    });
                }
            } else if (StreamURL.indexOf("https://www.twitch.tv/popout/") === 0) {
                StreamURL = StreamURL.replace("https://www.twitch.tv/popout/", "");
                if (StreamURL.indexOf("/") !== -1) {
                    StreamURL = StreamURL.slice(0, StreamURL.indexOf("/"));
                }
                StreamURL = `TW_${StreamURL}`;
                if (this.ActiveChat.filter((e) => e.text === StreamURL).length === 0) {
                    this.ActiveChat.push({
                        text: StreamURL,
                        IFrameEle: undefined,
                    });
                }
            } else if (StreamURL.indexOf("https://www.twitch.tv/") === 0) {
                StreamURL = StreamURL.replace("https://www.twitch.tv/", "");
                if (StreamURL.indexOf("/") === -1) {
                    if (StreamURL.indexOf("?") !== -1) {
                        StreamURL = StreamURL.slice(0, StreamURL.indexOf("?"));
                    }
                    StreamURL = `TW_${StreamURL}`;
                    if (this.ActiveChat.filter((e) => e.text === StreamURL).length === 0) {
                        this.ActiveChat.push({
                            text: StreamURL,
                            IFrameEle: undefined,
                        });
                    }
                }
            }
        },
        //= ====================== ACTIVE CHAT CONTROLLER ======================
        ColourPickerClose() {
            if (this.ColourPick === 1) {
                this.Profile[this.ProfileIdx].CC = this.ColourTemp;
            } else if (this.ColourPick === 2) {
                this.Profile[this.ProfileIdx].OC = this.ColourTemp;
            }
            this.ColourDialogue = false;
        },
        ColourPickerOK() {
            this.ColourDialogue = false;
        },
    },
};
</script>

<style>
.TopMenu {
  width:100%;
  position:absolute;
  top:0px;
  left:0px;
  z-index: 1;
}
.ChatOuterContainer{
  overflow-y: auto;
}
.ChatInnerContainer{
  position: absolute;
  bottom: 0px;
  width: 100%;
}
.ColourButton {
  margin-top: 19px;
  margin-left: 5px;
}
.ProfileListCard {
  position: absolute;
  bottom: 5px;
  right: 5px;
}
.ChatPanelContainer{
  display: grid;
  grid-auto-flow: column;
}
.ActiveChatIFrame{
  width: 100%;
  height: 100%;
}
</style>
