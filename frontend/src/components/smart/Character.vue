<template>
   <div class="row ml-0 mr-0">
    <!-- Character Image -->
    <div class="col col-md-4">
      <div class="characterWrapper">
        <div
          v-bind:class="
            ['character-cosmetic-applied-'+
              characterCosmetics[currentCharacterId],
              'character-animation-applied-'+
              characterCosmetics[currentCharacterId]
            ]
          "
        >
        <div class="animation" />
        <img
          class="characterImg placeholder"
          :src="getCharacterArt(characters[currentCharacterId])"
          :alt="getCharacterName(currentCharacterId)"
        />
        </div>
      </div>
    </div>
    <div class="col col-md-8">
      <!-- Character Name Heading -->
      <div class="mb-5 d-flex">
        <h1 class="title text-primary mb-0">{{ getCharacterName(currentCharacterId) }}</h1>
        <button class="edit-icon" @click="openChangeNameModal" ><img src="../../assets/edit-icon.svg" /></button>
        <span class="ml-auto align-self-end">#{{currentCharacterId}}</span>
      </div>
      <!-- Character chart info -->
      <div class="row mb-5 text-primary">
        <div class="col cell">
          <div class="table-bg"></div>
          <span class="main-font cell-title text-white">{{$t(`Character.level`)}}</span>
          <span class="alt-text cell-value">{{ characterLvl }}</span>
        </div>
        <div class="w-100 d-block d-md-none"></div>
        <div class="col cell">
          <div class="table-bg"></div>
          <span class="main-font cell-title text-white">{{$t(`Character.power`)}}</span>
          <span class="alt-text cell-value">{{ totalCharacterPower }}</span>
        </div>
        <div class="w-100 d-block d-md-none"></div>
        <div class="col cell" v-if="reputationLevelRequirements">
          <div class="table-bg"></div>
          <span class="main-font cell-title text-white">{{$t(`Character.reputation`)}}</span>
          <span class="cell-value">{{$t(`quests.reputationTier.${ReputationTier[getReputationLevel(reputation)]}`)}}</span>
        </div>
        <div class="w-100 d-block d-md-none"></div>
        <div class="col cell">
          <div class="table-bg"></div>
          <span class="main-font cell-title text-white">{{$t(`Character.element`)}}</span>
          <p class="alt-text cell-value">
            <span :class="characterTrait.toLowerCase() + '-icon circle-element'"></span>
            {{characterTrait}}
          </p>
        </div>
      </div>
        <!-- EXP STAMINA -->
      <div>
        <div class="mb-4 main-font text">
          <p class="mb-1 text-primary">
            <span class="text-white">
            {{$t(`Character.exp`)}}:{{" "}}
            </span>
              {{characterExp}}/{{RequiredXp(characterLvl)}}
          {{" "}}
            <span class="text-muted">({{$t(`Character.expNeeded`)}}{{" "}}{{characterLvl + 1}}: {{expDiff}})
            </span>
          </p>
          <b-progress class="progress-custom">
            <b-progress-bar
              class="bar"
              :value="expPercentage"
            ></b-progress-bar>
          </b-progress>
        </div>
        <div>
          <p class="mb-1 text-primary">
          <span class="text-white"> {{$t(`Character.stamina`)}}:{{" "}}</span>
          {{ getCharacterStamina(currentCharacterId) }}/200</p>
          <b-progress class="progress-custom">
            <b-progress-bar
              class="bar"
              :value="getCharacterStamina(currentCharacterId)/2"
            ></b-progress-bar>
          </b-progress>
        </div>
      </div>
      <!-- Character Tabs -->
      <div>
        <b-tabs pills fill nav-wrapper-class="mt-5 mb-4" >
          <upgrade-tab :soulBalance="soulBalance" @fetchSoulBalance="refreshData" />
          <skins-tab :availableSkins="availableSkins" @loadCosmeticsCount="loadCosmeticsCount" />
          <options-tab @openTransferModal="openTransferModal" @onSendToGarrison="onSendToGarrison" @openChangeTrait="openChangeTrait"
          @openTransferSoulModal="openTransferSoulModal" />
          <b-tab  disabled title-item-class="character-wrapper" title-link-class="character-tab">{{" "}}</b-tab>
        </b-tabs>
      </div>
    </div>
    <!--Character Change Trait Modal -->
    <b-modal @hide="removeErrors" class="centered-modal" ref="character-change-trait-modal"
      centered :content-class="isMobile() ? 'character-modal character-modal-mobile' : 'character-modal'" hide-footer hide-header-close
      dialog-class="dialog-character" size="lg">
      <h3 class="confirmation-title">  {{$t('Character.changeTrait')}}</h3>
      <span class="trait-pick">
      {{$t('characterList.pickTrait')}}
      </span>
      <div class="input mt-2">
        <select class="form-control" v-model="targetTrait" :disabled="availableTraits.length === 0">
          <option class="text-body" v-if="availableTraits.length === 0" value="">{{ $t('Character.noTraits') }}</option>
          <option class="text-body" v-else value="" disabled selected hidden>Please select a trait</option>
          <option class="text-body" v-for="x in availableTraits" :value="x" :key="x">{{ x }}</option>
        </select>
        <div class="inputImage" v-if="targetTrait">
          <img :src="targetTrait ? require(`@/assets/elements/${targetTrait}_Potion.png`) : require(`@/assets/elements/Lightning_Potion.png`)" />
          <span class="main-font">{{currentTraitTotal()}}</span>
        </div>
      </div>
      <div class="transferResultContainer">
        <small class="resultMsg text-center"> {{resultMsg}} </small>
      </div>
      <button class="modal-btn" :disabled="isSending || targetTrait === ''" @click="changeCharacterTraitCall">
          <span v-if="isSending"><i class="fas fa-spinner fa-spin"></i> Loading</span>
          <span v-else>Change</span>
      </button>
      <div class="footer-close" @click="$refs['character-change-trait-modal'].hide()">
        <p class="tapAny mt-4">{{$t('tapAnyWhere')}}</p>
        <p class="close-icon"></p>
      </div>
    </b-modal>
    <!-- Character Transfer Modal -->
    <b-modal class="centered-modal" ref="character-transfer-modal"
      centered :content-class="isMobile() ? 'character-modal character-modal-mobile' : 'character-modal'" hide-footer hide-header-close
      dialog-class="dialog-character" size="lg">
        <h3 class="confirmation-title">{{$t('Character.transfer')}}</h3>
        <b-form-input class="input" placeholder="Enter address" v-model="receiverAddress"/>
        <div class="transferResultContainer">
          <div class="loader" v-if="isSending">
            <i class="fas fa-spinner fa-spin"></i>
              Loading...
          </div>
          <span class="resultMsg text-center"> {{resultMsg}} </span>
        </div>
        <button :disabled="isSending || receiverAddress === ''" @click="transfer">Transfer</button>
        <div class="footer-close" @click="$refs['character-transfer-modal'].hide()">
          <p class="tapAny mt-4">{{$t('tapAnyWhere')}}</p>
          <p class="close-icon"></p>
        </div>
    </b-modal>
    <!-- Character Soul Transfer Modal -->
    <b-modal class="centered-modal" ref="character-transfer-soul-modal"
      centered :content-class="isMobile() ? 'character-modal character-modal-mobile' : 'character-modal'" hide-footer hide-header-close
      dialog-class="dialog-character" size="lg">
        <h3 class="confirmation-title">{{$t('Character.transferSoul')}}</h3>
        <div class="d-flex flex-column">
          <div class="row d-flex justify-content-between align-items-center mb-4">
            <div class="col col-md-3 d-flex justify-content-center align-items-center">
              <div class="soul-container mr-2">
                <img :src="require('@/assets/dusts/soulIcon.svg')" alt="soul"/>
              </div>
              <div class="character-text">
                <p class="mb-0 text-white soul-title">{{$t(`Character.soulTransferLabel`)}}</p>
                <p class="mb-0">{{ soulBalance }}</p>
              </div>
            </div>
            <div class="w-col col-md-6 d-flex flex-column">
              <input
                v-model="soulAmountToTransfer"
                class="range-character py-3"
                type="range"
                min="0"
                :max="soulBalance"
                :disabled="soulBalance <= 0"
                steps="10"
              />
            </div>
            <div class="col col-md-3 character-text d-flex justify-content-center align-items-center">
              <input id="powerAmount" type="number" v-model="soulAmountToTransfer" :min="0" :max="soulBalance"/>
              <button class="mx-1 px-2"  @click="handleMax">{{$t(`Character.max`)}}</button>
            </div>
          </div>
          <div class="custom-to mb-4 text-center">
            <h5>to</h5>
          </div>
          <div>
            <b-form-input class="input" placeholder="Enter address" v-model="receiverAddress"/>
            <div class="transferResultContainer">
              <div class="loader" v-if="isSending">
                <i class="fas fa-spinner fa-spin"></i>
                  Loading...
              </div>
              <span class="resultMsg text-center"> {{resultMsg}} </span>
            </div>
          </div>
        </div>
        <button :disabled="isSending || receiverAddress === ''" @click="onSoulTransferConfirm">Transfer</button>
        <div class="footer-close" @click="$refs['character-transfer-soul-modal'].hide()">
          <p class="tapAny mt-4">{{$t('tapAnyWhere')}}</p>
          <p class="close-icon"></p>
        </div>
    </b-modal>
    <!-- Character Change Name Modal -->
    <b-modal class="centered-modal" ref="character-change-name-modal"
      centered :content-class="isMobile() ? 'character-modal character-modal-mobile' : 'character-modal'" hide-footer hide-header-close
      dialog-class="dialog-character" size="lg">
      <template #modal-title>
        {{$t('Character.changeName')}}
      </template>
        <div class="input">
          <b-form-input placeholder="Enter new name" v-model="newName"/>
          <div class="inputImage">
            <img src="../../assets/elements/scroll_06_te.png" />
            <span class="main-font">{{haveRename}}/1</span>
          </div>
        </div>
        <div class="transferResultContainer">
          <div class="loader" v-if="isSending">
            <i class="fas fa-spinner fa-spin"></i>
              Loading...
          </div>
          <span class="resultMsg text-center"> {{resultMsg}} </span>
        </div>
        <button :disabled="isSending || newName === ''" @click="renameCharacterCall">Change</button>
        <div class="footer-close" @click="$refs['character-change-name-modal'].hide()">
          <p class="tapAny mt-4">{{$t('tapAnyWhere')}}</p>
          <p class="close-icon"></p>
        </div>
    </b-modal>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import { mapState, mapGetters, mapActions } from 'vuex';
import { BModal } from 'bootstrap-vue';

import SkinsTab from '@/components/smart/CharacterTabs/SkinsTab.vue';
import OptionsTab from '@/components/smart/CharacterTabs/OptionsTab.vue';
import UpgradeTab from '@/components/smart/CharacterTabs/UpgradeTab.vue';
import { Nft } from '@/interfaces/Nft';
import { getCharacterArt } from '@/character-arts-placeholder';
import { Quest, ReputationLevelRequirements, ReputationTier } from '@/views/Quests.vue';
import { CharacterTrait, RequiredXp } from '@/interfaces';
import { isValidWeb3Address } from '@/utils/common';


interface Skin {
  id: number;
  name?: string;
  amount: number;
}


interface Data {
  ReputationTier: typeof ReputationTier;
  reputationLevelRequirements?: ReputationLevelRequirements;
  soulBalance: number;
  powerAmount: number;
  haveRename: number;
  targetTrait: string;
  haveChangeTraitFire: number;
  haveChangeTraitEarth: number;
  haveChangeTraitWater: number;
  haveChangeTraitLightning: number;
  quest: null | Quest;
  haveCharacterCosmetics: Skin[]
  isSending: boolean
  resultMsg: string
  receiverAddress: string
  newName: string,
  soulToTransfer: number,
  soulAmountToTransfer: number
}

interface StoreMappedActions {
  getReputationLevelRequirements(): Promise<ReputationLevelRequirements>;
  fetchSoulBalance(): Promise<number>;
  getCharacterQuestData(payload: { characterId: string | number }): Promise<Quest>;
  fetchTotalRenameTags(): Promise<number>;
  fetchTotalCharacterFireTraitChanges(): Promise<number>;
  fetchTotalCharacterEarthTraitChanges(): Promise<number>;
  fetchTotalCharacterWaterTraitChanges(): Promise<number>;
  fetchTotalCharacterLightningTraitChanges(): Promise<number>;
  changeCharacterTraitFire(payload: {id: string}): Promise<void>;
  changeCharacterTraitWater(payload: {id: string}): Promise<void>;
  changeCharacterTraitEarth(payload: {id: string}): Promise<void>;
  changeCharacterTraitLightning(payload: {id: string}): Promise<void>;
  renameCharacter(payload: {id: string; name: string;}): Promise<void>;
  fetchOwnedCharacterCosmetics(payload: {cosmetic: string}): Promise<number>;
  getCharacterStamina(payload: string): Promise<number>;
  transferNFT(payload: {
    nftId: number;
    receiverAddress: string;
    nftType: string;
  }): Promise<void>;
  sendToGarrison(id: string): Promise<void>;
  transferSoul(payload: {targetAddress: string, soulAmount: number}): Promise<void>;
}

export default Vue.extend({
  components: { UpgradeTab, OptionsTab, SkinsTab },
  data(): Data{
    return {   ReputationTier,
      reputationLevelRequirements: undefined,
      soulBalance: 0,
      powerAmount: 0,
      haveRename: 0,
      targetTrait: '',
      haveChangeTraitFire: 0,
      haveChangeTraitEarth: 0,
      haveChangeTraitWater: 0,
      haveChangeTraitLightning: 0,
      quest: null,
      haveCharacterCosmetics: [{
        id: 0,
        name: 'Default',
        amount: 1
      }],
      isSending: false,
      resultMsg: '',
      receiverAddress: '',
      newName: '',
      soulToTransfer: 0,
      soulAmountToTransfer: 0,
    };
  },
  computed: {
    ...mapState([
      'characterCosmetics',
      'currentCharacterId',
      'characters',
      'characterStaminas',
      'ownedGarrisonCharacterIds',
    ]),
    ...mapGetters(['getCharacterName', 'getCharacterPower', 'getCharacterStamina']),
    availableTraits(): string[] {
      const availableTraits = [];
      if(this.haveChangeTraitFire > 0) {
        availableTraits.push('Fire');
      }
      if(this.haveChangeTraitEarth > 0) {
        availableTraits.push('Earth');
      }
      if(this.haveChangeTraitWater > 0) {
        availableTraits.push('Water');
      }
      if(this.haveChangeTraitLightning > 0) {
        availableTraits.push('Lightning');
      }

      return availableTraits;
    },
    totalTraits(): number {
      return +this.haveChangeTraitFire + +this.haveChangeTraitEarth + +this.haveChangeTraitWater + +this.haveChangeTraitLightning;
    },
    reputation(): number {
      return this.quest?.reputation ?? 0;
    },
    characterTrait(): string {
      const characterWithId = this.characters[this.currentCharacterId];
      return CharacterTrait[characterWithId?.trait] ?? '';
    },
    selectedCharacter(): Nft{
      return this.characters[this.currentCharacterId];
    },
    characterStamina(): number {
      return this.ownedGarrisonCharacterIds.includes(this.currentCharacterId)
        ? this.characterStaminas[this.currentCharacterId]
        : this.timestampToStamina(this.characters[this.currentCharacterId]?.staminaTimestamp ?? 0);
    },
    characterExp(): number {
      return this.characters[this.currentCharacterId]?.xp ?? 0;
    },
    expDiff(): string {
      const result = RequiredXp(this.characters[this.currentCharacterId]?.level + 1 ?? 0) - (this.characters[this.currentCharacterId]?.xp ?? 0);
      return new Intl.NumberFormat().format(result);
    },
    expPercentage(): number {
      const total = ((this.characters[this.currentCharacterId]?.xp ?? 0)/ RequiredXp(this.characters[this.currentCharacterId]?.level ?? 0) ?? 1) * 100;
      return total;
    },
    availableSkins(): Skin[] {
      const availableSkins = [];

      const currentCosmetic = this.characterCosmetics[this.currentCharacterId];

      if (+currentCosmetic > 0) {
        availableSkins.push({
          id: currentCosmetic,
          amount: 1
        });
      }

      for(let i = 0; i < 19; i++) {
        if(+this.haveCharacterCosmetics[i]?.amount > 0 && !availableSkins.find(item=> +item.id === i)) {
          availableSkins.push(this.haveCharacterCosmetics[i]);
        }
      }
      return availableSkins;
    },
    characterLvl(): number {
      return this.characters[this.currentCharacterId]?.level + 1 ?? 1;
    },
    totalCharacterPower(): number {
      return this.getCharacterPower(this.currentCharacterId);
    },
  },
  methods: {
    ...mapActions([
      'getReputationLevelRequirements',
      'getCharacterQuestData',
      'fetchSoulBalance',
      'fetchTotalCharacterFireTraitChanges',
      'fetchTotalCharacterEarthTraitChanges',
      'fetchTotalCharacterWaterTraitChanges',
      'fetchTotalCharacterLightningTraitChanges',
      'changeCharacterTraitFire',
      'changeCharacterTraitWater',
      'changeCharacterTraitEarth',
      'changeCharacterTraitLightning',
      'transferNFT',
      'renameCharacter',
      'sendToGarrison',
      'fetchOwnedCharacterCosmetics',
      'fetchTotalRenameTags',
      'transferSoul',
    ]) as StoreMappedActions,
    getCharacterArt,
    RequiredXp,
    removeErrors(){
      this.resultMsg = '';
    },
    async onSoulTransferConfirm() {
      if(!isValidWeb3Address(this.receiverAddress) || this.soulAmountToTransfer === 0) return;
      this.isSending = true;
      await this.transferSoul({ targetAddress: this.receiverAddress, soulAmount: this.soulAmountToTransfer });
      this.isSending = false;
      this.soulBalance = await this.fetchSoulBalance();
      this.soulAmountToTransfer = 0;
      this.receiverAddress = '';
      (this.$refs['character-transfer-soul-modal'] as BModal).hide();
    },
    currentTraitTotal() {
      switch(this.targetTrait){
      case 'Fire':
        return +this.haveChangeTraitFire;
      case 'Earth':
        return +this.haveChangeTraitEarth;
      case 'Water':
        return +this.haveChangeTraitWater;
      case 'Lightning':
        return +this.haveChangeTraitLightning;
      default:
        return 0;
      }
    },
    handleMax(){
      this.soulAmountToTransfer = this.soulBalance;
    },
    openTransferSoulModal(){
      (this.$refs['character-transfer-soul-modal'] as BModal).show();
    },
    openTransferModal(){
      (this.$refs['character-transfer-modal'] as BModal).show();
    },
    openChangeTrait(){
      (this.$refs['character-change-trait-modal'] as BModal).show();
    },
    async openChangeNameModal() {
      this.haveRename = await this.fetchTotalRenameTags();
      (this.$refs['character-change-name-modal'] as BModal).show();
    },
    getReputationLevel(reputation: number) {
      if (!this.reputationLevelRequirements) return;
      if (reputation < this.reputationLevelRequirements.level2) {
        return ReputationTier.PEASANT;
      } else if (reputation < this.reputationLevelRequirements.level3) {
        return ReputationTier.TRADESMAN;
      } else if (reputation < this.reputationLevelRequirements.level4) {
        return ReputationTier.NOBLE;
      } else if (reputation < this.reputationLevelRequirements.level5) {
        return ReputationTier.KNIGHT;
      } else {
        return ReputationTier.KING;
      }
    },
    timestampToStamina(timestamp: number): number {
      if(timestamp > Math.floor(Date.now()/1000)) return 0;
      return +Math.min((Math.floor(Date.now()/1000) - timestamp) / 300, 200).toFixed(0);
    },
    async refreshData(){
      this.reputationLevelRequirements =  await this.getReputationLevelRequirements();
      this.soulBalance = +(await this.fetchSoulBalance());
    },
    async fetchCharacterQuestData(){
      this.quest = await this.getCharacterQuestData({characterId: this.currentCharacterId});
    },

    async loadConsumablesCount() {
      this.haveChangeTraitFire = await this.fetchTotalCharacterFireTraitChanges();
      this.haveChangeTraitEarth = await this.fetchTotalCharacterEarthTraitChanges();
      this.haveChangeTraitWater = await this.fetchTotalCharacterWaterTraitChanges();
      this.haveChangeTraitLightning = await this.fetchTotalCharacterLightningTraitChanges();
    },
    async loadCosmeticsCount() {
      for(let i = 1; i < 19; i++) {
        const amount = await this.fetchOwnedCharacterCosmetics({cosmetic: i.toString()});
        this.haveCharacterCosmetics.push({
          id: i,
          amount: +amount
        });
      }
    },
    async changeCharacterTraitCall(bvModalEvt: Event) {
      if(!this.targetTrait) {
        bvModalEvt.preventDefault();
      }
      this.isSending = true;
      try{
        switch(this.targetTrait) {
        case 'Fire':
          await this.changeCharacterTraitFire({ id: this.currentCharacterId });
          this.haveChangeTraitFire = await this.fetchTotalCharacterFireTraitChanges();
          break;
        case 'Earth' :
          await this.changeCharacterTraitEarth({ id: this.currentCharacterId });
          this.haveChangeTraitEarth = await this.fetchTotalCharacterEarthTraitChanges();
          break;
        case 'Water':
          await this.changeCharacterTraitWater({ id: this.currentCharacterId });
          this.haveChangeTraitWater = await this.fetchTotalCharacterWaterTraitChanges();
          break;
        case 'Lightning':
          await this.changeCharacterTraitLightning({ id: this.currentCharacterId });
          this.haveChangeTraitLightning = await this.fetchTotalCharacterLightningTraitChanges();
          break;
        }
      }catch(e: any){
        if(e.code as number === 4001) this.resultMsg = (this as any).$t('Character.cancelledTransaction');
        else this.resultMsg = (this as any).$t('Character.changeTraitError');
        this.isSending = false;
        return;
      }
      this.isSending = false;
      this.resultMsg = '';
      this.targetTrait = '';
      (this.$refs['character-change-trait-modal'] as BModal).hide();
    },
    async transfer(bvModalEvt: Event) {
      bvModalEvt.preventDefault();
      this.isSending = true;
      try {
        await this.transferNFT({
          nftId: this.currentCharacterId,
          receiverAddress: this.receiverAddress,
          nftType: 'character'
        });
      }
      catch(e: any) {
        if(e.code as number === 4001) this.resultMsg = 'You cancelled the transaction.';
        else this.resultMsg = 'Error while transferring your ' + 'character' + ' to ' + this.receiverAddress;
      }
      this.isSending = false;
    },
    async renameCharacterCall(bvModalEvt: Event) {
      if (!this.haveRename) {
        (this.$refs['character-change-name-modal'] as BModal).hide();
        return;
      }
      if(this.newName.length < 2 || this.newName.length > 24){
        bvModalEvt.preventDefault();
        return;
      }

      await this.renameCharacter({id: this.currentCharacterId, name: this.newName.trim()});
      this.haveRename = await this.fetchTotalRenameTags();
      (this.$refs['character-change-name-modal'] as BModal).hide();
    },
    async onSendToGarrison() {
      await this.sendToGarrison(this.currentCharacterId);
    },
  },
  watch: {
    async selectedCharacter(newValue){
      if (newValue) {
        await this.fetchCharacterQuestData();
        await this.refreshData();
      }
    },
  },
  async mounted(){
    await this.refreshData();
    await this.fetchCharacterQuestData();
    await this.loadConsumablesCount();
  },
});
</script>

<style lang="scss" scoped>
@import '../../styles/character-cosmetics.css';
.modal-btn{
  font-family: Roboto;
}
.custom-to{
  color: #9e8a57 !important;
  text-transform: uppercase;
  font: normal normal bold 30px/38px Trajan;
  width: 100%;
}
.character-text{
  color: #7F8693;
}

.character-text input {
  background: #000E1D 0% 0% no-repeat padding-box;
  border: 1px solid #404857;
  border-radius: 3px;
  color: #fff;
  width: 53px;
  height: 32px;
  text-align: center;
}

.character-text button {
  background: #1E293C 0% 0% no-repeat padding-box !important;
  border: 1px solid #1E293C !important;
  border-radius: 3px !important;
  color: #fff !important;
  height: 32px !important;
  margin-top: 0px !important;
}
/* Chrome, Safari, Edge, Opera */
.character-text input::-webkit-outer-spin-button,
.character-text input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/* Firefox */
.character-text input[type=number] {
  -moz-appearance: textfield;
}

.character-container {
  display: grid;
  grid-template-columns: 2fr 3fr;
  gap: 1.5rem;
}


.characterWrapper {
  position: relative;
  width: 100%;
  max-width: 500px;
  max-height: 100vh;
  height: max-content;
  background-image: url("../../assets/placeholder/standImage.png");
  background-size: contain;
  background-position-y: bottom;
  background-repeat: no-repeat;
  margin-top: 5rem;
  padding-bottom: 3rem;
}
.characterImg {
  width: 100%;
  object-fit: contain;
  max-height: 60vh;
  max-width: 100%;
  transform: none;
}


.title {
  text-transform: uppercase;
  font-family: 'Trajan', serif;
  font-weight: 700;
  line-height: 38px;
  font-size: 30px;
}

.trait-pick{
  font-family: Roboto;
}

.edit-icon {
  background: transparent;
  border: none;
  outline: none;
  margin-left: 5px;
  display: flex;
  padding-top: 10px;
}

.edit-icon img {
  width: 1rem;
  height: 1rem;
}


.text {
  line-height: 18px;
}


.progress-custom {
  height: 5px;
}

.confirmation-title{
  color: #EDCD90;
  text-transform: uppercase;
  margin-bottom: 1em;
}

.bar {
  background-color: #D01111;
}

.tabs {
  min-height: 400px;
}


.cell {
  display: flex;
  flex-direction: column;
  text-align: center;
  border: 1px solid #404857;
  padding: 1.5rem 2rem;
  position: relative;
}

.cell span, .cell p {
  position: relative;
  z-index: 1;
}

.table-bg{
  position: absolute;
  width: 100%;
  height: 100%;
  top:0;
  left: 0;
  background: #1E293C;
  opacity: 0.2;
  z-index: 1;
}

.cell:first-child{
  border-top-left-radius: 10px;
  border-bottom-left-radius: 10px;
}

.cell:last-child{
  border-top-right-radius: 10px;
  border-bottom-right-radius: 10px;
}

.cell-title{
  font-size: 16px;
  line-height: 21px;
  letter-spacing: 0;
}


.cell-value {
  font-size: 18px;
  line-height: 26px;
  text-transform: uppercase;
  margin: 0;
}




.offset {
  position: absolute;
  display: flex;
  flex-direction: column;
  text-align:center;
  width: 100%;
  bottom: 0;
  transform: translateY(140px);
  color: #fff;
  background: transparent!important;
  border: none;
  outline: none;
  img {
    width: 40px;
    height: 40px;
    margin: 0 auto;
  }
}


.input img {
  width: 40px;
  height: 40px;
  align-self: center;
}

.input input, .input select {
  background: transparent;
  border: none;
  height: 70px;
  outline: none;
  color: #fff;
}

.inputImage {
  position: relative;
  margin: 12px;
}

.inputImage span {
  position: absolute;
  bottom: 0;
  right: 0;
  font-size: 12px;
  line-height: 16px;
  color: #B9BFCC;
}


@media all and (max-width: 600px) {
  .characterWrapper{
    margin-top: 1rem;
  }
}

</style>>

