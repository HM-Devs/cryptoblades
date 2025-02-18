
<template>
  <div class="position-relative background-image">
    <div class="blind-background position-absolute w-100 h-100 opacity-75"></div>
    <div v-if="!haveCharacters" class="blank-slate position-relative content">
      <div class="initial-recruitment">
        <div class="tob-bg-img promotion-decoration">
          <img class="vertical-decoration bottom" src="../assets/border-element.png">
        </div>
        <strong class="upper-text">{{ $t("plaza.welcome") }}</strong>
        <div class="bot-bg-img promotion-decoration">
            <img src="../assets/border-element.png">
        </div>
        <big-button
        class="button mt-5"
        :mainText="$t('plaza.recruitCharacter') + ` ${recruitCost} SKILL`"
        :disabled="!canRecruit()"
        @click="onMintCharacter"
        tagname="recruit_character"
      />
      </div>
      <div v-if="formatSkill() < recruitCost" >
        <br>
        <i18n path="plaza.notEnoughSkill" tag="label" for="plaza.notEnoughSkillLink">
          <a v-bind:href="`${getExchangeUrl}`" target="_blank" rel="noopener noreferrer">{{$t("plaza.notEnoughSkillLink")}}</a>
        </i18n>
        <a :href="getExchangeTransakUrl()" target="_blank" rel="noopener noreferrer"> {{$t("plaza.buyBNBTransak")}}</a>.
      </div>
    </div>
    <div v-else class="char-content">
      <CharacterNav
        :activeTab="activeTab"
        :havePlazaCharacters="havePlazaCharacters"
        @toggle="toggleGarrison"
        :recruitCost="recruitCost"
        :soulBalance="soulBalance"
        :ownCharacters="ownCharacters"
        @mintCharacter="onMintCharacter"
        @onClaimGarrisonXp="onClaimGarrisonXp"
        @changeTab="onChangeTab"
      />
      <template v-if="activeTab === 'info' && havePlazaCharacters">
        <div class="d-flex justify-content-end pr-5 pt-4 slippage-checkbox" v-if="ownCharacters.length === 4">
          <b-checkbox variant="primary" v-model="mintSlippageApproved">
            <span><b>{{$t('plaza.approveMintSlippage')}}</b></span>
            <b-icon-question-circle class="ml-1 centered-icon" v-tooltip.bottom="$t('plaza.dynamicPricesDetails',
              { decreaseAmount: mintPriceDecreasePerHour, increaseAmount: mintCharacterPriceIncrease, minimumPrice: mintCharacterMinPrice})"/>
          </b-checkbox>
        </div>
        <character class="char-info" />
      </template>
      <template v-if="activeTab === 'garrison'">
        <div class="row mt-3 z-index-1 char-info ml-0 mr-0">
          <div class="col">
            <div>
              <div class="d-flex flex-column flex-md-row justify-content-space-between garrisson-content">
                <h1>{{$t('characters')}} ({{ ownedGarrisonCharacterIds.length }})</h1>
              </div>
              <character-list
                :showNftOptions="true"
                :isGarrison="true"
                @input="setCurrentCharacter"
              />
            </div>
          </div>
        </div>
      </template>
      <template v-if="activeTab === 'burn'">
        <!-- Character Burn -->
        <div class="pt-5 char-info">
          <div>
            <div class="col-md-12">
              <div class="row mobile-flip">
                <div class="col-md-4 character-container" >
                  <div class="magic-circle">
                    <div class="spinning-circle"></div>
                    <div class="soul-container">
                      <div class="soul-img" :class="glowImage ? 'glowUp' : ''"></div>
                      <p>+{{totalSoul.toLocaleString()}}</p>
                    </div>
                  </div>
                  <div class="btn-container">
                      <div class="ml-3 mt-4 mt-md-0 ml-md-auto recruit-btn text-uppercase"
                      :style="
                        burnCharacterIds.length === 0 ||
                        powerLimitExceeded ||
                        (burnOption === 1 && !targetCharacterId) ||
                        !canBurn() ||
                        (burnCharacterIds.length > 1 && disableCharacterBurn) ||
                        isBurnInProgress ? 'opacity: 0.5' : ''"
                      v-tooltip=" (burnCharacterIds.length > 1 && disableCharacterBurn) ? $t('plaza.disableDynamicMinting') : $t('plaza.burnSelected')"
                      @click="showBurnConfirmation">
                        <span v-if="isBurnInProgress" class="gtag-link-others custom-recruit-text"> {{$t('plaza.burning')}}</span>
                        <span v-else class="gtag-link-others custom-recruit-text"> <p>{{$t('plaza.burn')}}</p>  {{burnCost }} SKILL</span>
                      </div>
                  </div>
                  <div class="scroll" v-if="isMobile()">
                    <h5>{{$t('Character.scrollDown')}}</h5>
                    <div>
                      <p> &#8811;</p>
                    </div>
                  </div>
                </div>
                <div class="col-md-8 character-container" >
                  <character-list
                  :showFilters="true"
                  :showGivenCharacterIds="true"
                  :toBurn="burnCharacterIds"
                  :characterIds="remainingCharactersIds"
                  @input="addBurnCharacter"/>
                </div>
              </div>
            </div>
          </div>
        </div>

      </template>
    </div>

    <b-modal centered hide-header hide-footer class="centered-modal text-center"
      id="burn-confirmation-modal" ref="burn-confirmation-modal"
      :title="$t('plaza.burnConfirmation')">
      <h3 class="confirmation-title">{{$t('plaza.burnConfirmation')}}</h3>
      <div class="text-center burn-content mt-4">
        <b-icon icon="exclamation-circle" variant="warning" />
        {{ $t('plaza.burnWarning', { characterAmount: burnCharacterIds.length })}}<br>
        {{ $t('plaza.cantBeUndone')}}
      </div>
      <div class="text-center burn-content">
        <b-icon icon="exclamation-circle" variant="warning" />
        {{ $t('plaza.noRefunds')}}
      </div>
       <div class="footer-btn">
        <button class="close-btn" :disabled="burnCharacterIds.length === 0" @click="onBurnConfirm">
          {{isBurnInProgress ? 'Burning..' : $t('blacksmith.confirm')}}</button>
      </div>
      <div class="footer-close" @click="$refs['burn-confirmation-modal'].hide()">
        <p class="tapAny mt-4">{{$t('tapAnyWhere')}}</p>
        <p class="close-icon"></p>
      </div>
    </b-modal>
    <div v-if="showAds && !isMobile()" class="ad-container align-items-center">
      <script2 async src="https://coinzillatag.com/lib/sticky.js"></script2>
        <div class="coinzilla" data-zone="C-2621de2f7c4f7a272"></div>
        <script2>window.coinzilla_sticky = window.coinzilla_sticky
            || [];function czilla(){coinzilla_sticky.push(arguments);}czilla('2621de2f7c4f7a272');</script2>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import { mapActions, mapGetters, mapState, mapMutations } from 'vuex';
import { BModal } from 'bootstrap-vue';
import BN from 'bignumber.js';
import i18n from '@/i18n';
import { Nft } from '@/interfaces/Nft';
import { CharacterPower } from '@/interfaces';
import BigButton from '@/components/BigButton.vue';
import CharacterList from '@/components/smart/CharacterList.vue';
import CharacterNav from '@/components/CharacterNav.vue';
import Character from '@/components/smart/Character.vue';
import Events from '@/events';
import { getConfigValue } from '@/contracts';


import { fromWeiEther, toBN } from '../utils/common';
import {
  burningManager as featureFlagBurningManager
} from '../feature-flags';


interface StoreMappedActions {
  fetchMintCharacterFee(): Promise<string>;
  mintCharacter(value: boolean): Promise<void>;
  fetchMintCharacterPriceDecreasePerSecond(): Promise<number>;
  fetchCharacterMintIncreasePrice(): Promise<number>;
  fetchMintCharacterMinPrice(): Promise<number>;
  fetchSoulBalance(): Promise<string>;
  fetchBurnPowerMultiplier(): Promise<string>;
  fetchCharactersBurnCost(payload: string[]): Promise<string>;
  burnCharactersIntoSoul(payload: string[]): Promise<void>;
  burnCharactersIntoCharacter(payload: {burnIds: string[], targetId: string}): Promise<void>;
  claimGarrisonXp(payload: string[]): Promise<void>;
  fetchUsdSkillValue(payload: string | number): Promise<string | number>;
}

interface StoreMappedGetters {
  getExchangeTransakUrl(): string;
}

interface Data {
  recruitCost: string;
  showAds: boolean;
  updateInterval: ReturnType<typeof setInterval> | null;
  mintSlippageApproved: boolean;
  mintPriceDecreasePerHour: string;
  mintCharacterPriceIncrease: string;
  mintCharacterMinPrice: string;
  activeTab: string;
  soulBalance: number;
  disableCharacterBurn: boolean;
  burnCost: number;
  burnPowerMultiplier: number;
  burnCharacterIds: string[];
  remainingCharactersIds: string[];
  isUpgrading: boolean;
  isBurning: boolean;
  isTransferring: boolean;
  soulAmount: number;
  targetCharacterId: string;
  remainingPowerLimit: number;
  burnOption: number;
  isBurnInProgress: boolean;
  isClaimingXp: boolean;
  totalSoul: number;
  glowImage: boolean;
}

export default Vue.extend({
  data(): Data{
    return {
      activeTab: 'info',
      updateInterval: null as ReturnType<typeof setInterval> | null,
      recruitCost: '0',
      mintSlippageApproved: false,
      mintPriceDecreasePerHour: '0',
      mintCharacterPriceIncrease: '0',
      mintCharacterMinPrice: '0',
      showAds: false,
      soulBalance: 0,
      disableCharacterBurn: (getConfigValue('featureSupport').disableDynamicMinting),
      burnCost: 0,
      burnPowerMultiplier: 1,
      burnCharacterIds: [],
      remainingCharactersIds: [],
      isUpgrading: false,
      isBurning: false,
      isTransferring: false,
      soulAmount: 0,
      targetCharacterId: '',
      remainingPowerLimit: 0,
      burnOption: 0,
      isBurnInProgress: false,
      isClaimingXp: false,
      totalSoul: 0,
      glowImage: false
    };
  },
  computed: {
    ...mapState([
      'characters',
      'currentCharacterId',
      'maxStamina',
      'ownedGarrisonCharacterIds',
      'characterStaminas',
      'skillBalance',
      'skillRewards',
      'ownedCharacterIds',
      'xpRewards',
      'characterCosmetics',
    ]),
    ...mapGetters([
      'getCharacterName',
      'ownGarrisonCharacters',
      'getCharacterPower',
      'contracts',
      'ownCharacters',
      'getExchangeUrl',
      'getCharacterIsInArena'
    ]),
    selectedCharacter(): Nft{
      return this.characters[this.currentCharacterId];
    },
    burnPower(): number {
      let power = 0;
      if(!this.isUpgrading) {
        this.ownCharacters.map((x: { id: number; }) => {
          if(this.burnCharacterIds.includes(x.id.toString())) {
            power += this.getCharacterPower(x.id);
          }
        });
        this.ownGarrisonCharacters.map((x: { id: number; }) => {
          if(this.burnCharacterIds.includes(x.id.toString())) {
            power += this.getCharacterPower(x.id);
          }
        });
        power = Math.floor(power * this.burnPowerMultiplier);
      }
      else {
        power = this.soulAmount * 10;
      }

      return power;
    },
    powerLimitExceeded(): boolean {
      return (this.isUpgrading || this.burnOption === 1) && this.burnPower > this.remainingPowerLimit;
    },
    haveCharacters(): boolean {
      return this.ownedGarrisonCharacterIds.length > 0 || this.ownCharacters?.length > 0;
    },
    havePlazaCharacters(): boolean {
      return this.ownCharacters?.length > 0;
    },
    canClaimGarrisonXp(): boolean {
      return this.ownedGarrisonCharacterIds.filter((id: string|number) => +this.xpRewards[id] > 0).length > 0;
    },
    burningEnabled(): boolean {
      return featureFlagBurningManager;
    },
  },
  methods: {
    ...mapMutations(['setCurrentCharacter']),
    ...mapActions([
      'mintCharacter',
      'fetchMintCharacterPriceDecreasePerSecond',
      'fetchCharacterMintIncreasePrice',
      'fetchMintCharacterMinPrice',
      'fetchMintCharacterFee',
      'fetchSoulBalance',
      'fetchBurnPowerMultiplier',
      'fetchCharactersBurnCost',
      'burnCharactersIntoSoul',
      'burnCharactersIntoCharacter',
      'claimGarrisonXp',
      'fetchUsdSkillValue',
    ]) as StoreMappedActions,
    ...mapGetters(['getExchangeTransakUrl']) as StoreMappedGetters,

    onChangeTab(tab: string) {
      this.activeTab = tab;
    },
    toggleGarrison(tab: string) {
      if (this.activeTab === 'info' && this.ownedGarrisonCharacterIds.includes(this.currentCharacterId)) {
        this.setCurrentCharacter(this.ownedCharacterIds[0]);
      }

      this.activeTab = tab;
    },
    async onClaimGarrisonXp() {
      this.isClaimingXp = true;
      try {
        await this.claimGarrisonXp(this.ownedGarrisonCharacterIds.filter((id: string|number) => +this.xpRewards[id] > 0));
      }
      finally {
        this.isClaimingXp = true;
      }
    },
    async onMintCharacter() {
      try {
        await this.mintCharacter(this.mintSlippageApproved);
        await this.updateMintCharacterFee();
      } catch (e) {
        (this as any).$dialog.notify.error(i18n.t('plaza.couldNotMint'));
      }
    },
    async addBurnCharacter(id: number) {
      if(this.getCharacterIsInArena(id)) {
        (this as any).$dialog.notify.error(i18n.t('plaza.busyInArena'));
        return;
      }

      if(!this.burnCharacterIds.includes(id.toString())){
        this.burnCharacterIds.push(id.toString());
        this.totalSoul += this.getCharacterPower(id)/10;
      }else{
        this.burnCharacterIds = this.burnCharacterIds.filter(val => val !== id.toString());
        this.totalSoul -= this.getCharacterPower(id)/10;
      }
      await this.updateBurnCost();
    },
    async removeBurnCharacter(id: number) {
      this.burnCharacterIds = this.burnCharacterIds.filter(val => !val.includes(id.toString()));
      this.totalSoul -= this.getCharacterPower(id)/10;
    },
    canBurn() {
      const cost = toBN(this.burnCost);
      const balance = toBN(+fromWeiEther(this.skillBalance) + +fromWeiEther(this.skillRewards));
      return balance.isGreaterThanOrEqualTo(cost);
    },
    showBurnConfirmation() {
      if(!(this.burnCharacterIds.length === 0 ||  this.powerLimitExceeded || (this.burnOption === 1 && !this.targetCharacterId)
      || !this.canBurn() || this.isBurnInProgress || (this.burnCharacterIds.length > 1 && this.disableCharacterBurn))){
        (this.$refs['burn-confirmation-modal'] as BModal).show();
      }
    },
    async toggleSoulCreation() {
      this.soulBalance = +(await this.fetchSoulBalance());
      await this.updateBurnCost();
      this.burnPowerMultiplier = +fromWeiEther(await this.fetchBurnPowerMultiplier());
      if(this.activeTab === 'burn') {
        this.remainingCharactersIds = this.ownCharacters.map((x: { id: string; }) => x.id.toString()).concat(this.ownedGarrisonCharacterIds as string[]);
      }
      this.isUpgrading = false;
    },
    async selectAll(filterCharacter: any){
      await this.computeSoul(filterCharacter);
      await this.updateBurnCost();
    },
    computeSoul(filterCharacters: any){
      filterCharacters.forEach((filterCharacter: { id: { toString: () => string; }; })=> {
        if(!this.burnCharacterIds.includes(filterCharacter.id.toString())){
          this.burnCharacterIds.push(filterCharacter.id.toString());
          this.totalSoul += this.getCharacterPower(filterCharacter.id)/10;
        }
      });
    },
    clearAllBurn(){
      this.burnCharacterIds = [];
      this.remainingCharactersIds = (this.ownCharacters.map((x: { id: string; }) => x.id.toString())
        .concat(this.ownedGarrisonCharacterIds) as string[])
        .filter(x => x.toString() !== this.targetCharacterId);
      this.burnCost = 0;
      this.totalSoul = 0;
    },
    setMaxSoulAmount() {
      if (this.isTransferring) {
        this.soulAmount = this.soulBalance;
      } else {
        this.soulAmount = this.remainingPowerLimit > this.soulBalance * 10 ? this.soulBalance : this.remainingPowerLimit / 10;
      }
    },
    async updateBurnCost() {
      this.burnCost = this.burnCharacterIds.length > 0 ? +fromWeiEther(await this.fetchCharactersBurnCost(this.burnCharacterIds)) : 0;
    },
    checkStorage() {
      if (process.env.NODE_ENV === 'development') this.showAds = false;
      else this.showAds = localStorage.getItem('show-ads') === 'true';
    },
    canRecruit() {
      const cost = toBN(this.recruitCost);
      const balance = toBN(+fromWeiEther(this.skillBalance) + +fromWeiEther(this.skillRewards));
      return balance.isGreaterThanOrEqualTo(cost);
    },
    formatSkill() {
      return fromWeiEther(this.skillBalance);
    },
    async updateMintCharacterFee() {
      const recruitCost = await this.fetchMintCharacterFee();
      const skillRecruitCost = await this.fetchUsdSkillValue(recruitCost);
      this.recruitCost = new BN(skillRecruitCost).div(new BN(10).pow(18)).toFixed(4);
    },
    updatedRemainingPowerLimit() {
      const targetCharacter = this.ownCharacters.concat(this.ownGarrisonCharacters)
        .find((x: { id: any; }) => x.id.toString() === this.targetCharacterId.toString());
      this.remainingPowerLimit = 4 * CharacterPower(targetCharacter.level) - this.getCharacterPower(this.targetCharacterId.toString());
    },
    async onBurnConfirm() {
      if(this.burnCharacterIds.length === 0) return;
      this.isBurnInProgress = true;
      try {
        if(this.burnOption === 0) {
          // burning into soul
          await this.burnCharactersIntoSoul(this.burnCharacterIds);
        }
        else {
          // burning into character
          await this.burnCharactersIntoCharacter({ burnIds: this.burnCharacterIds, targetId: this.targetCharacterId });
          this.updatedRemainingPowerLimit();
        }
      }
      finally {
        (this.$refs['burn-confirmation-modal'] as BModal).hide();
        this.isBurnInProgress = false;
      }
      this.soulBalance = +(await this.fetchSoulBalance());
      this.burnCharacterIds = [];
      this.burnCost = 0;
      this.toggleSoulCreation();
    },
  },
  async mounted(){
    this.checkStorage();
    Events.$on('select-all', (filteredCharacters: any[])=>{
      this.selectAll(filteredCharacters);
    });
    Events.$on('deselect-all', async (deselectedIds: any[])=>{
      deselectedIds.forEach(deselectedId => {
        this.removeBurnCharacter(deselectedId.id);
      });

      await this.updateBurnCost();
    });
  },
  watch: {
    activeTab(){
      this.toggleSoulCreation();
    },
    async ownedCharacterIds(){
      await this.updateMintCharacterFee();
      this.toggleSoulCreation();
    },
    totalSoul(){
      this.glowImage = true;
      setTimeout(() => {
        this.glowImage = false;
      }, 500);
    }
  },
  async created(){
    this.mintPriceDecreasePerHour = new BN(await this.fetchMintCharacterPriceDecreasePerSecond()).div(new BN(10).pow(18)).multipliedBy(60*60).toFixed(6);
    this.mintCharacterPriceIncrease = new BN(await this.fetchCharacterMintIncreasePrice()).div(new BN(10).pow(18)).toFixed(6);
    this.mintCharacterMinPrice = new BN(await this.fetchMintCharacterMinPrice()).div(new BN(10).pow(18)).toFixed(4);
    this.updateMintCharacterFee();
  },
  components:{
    BigButton,
    CharacterList,
    CharacterNav,
    Character
  }
});
</script>
<style scoped lang="scss">

.background-image {
  background-image: url('../assets/artwork-fantasy-art-ruins.png') ;
  background-size: cover;
  background-repeat: no-repeat;
  background-position: top right;
  min-height: calc(100vh - 120px);
  height: 100%;
  z-index: 0;
}
.content {
  z-index: 10;
  position: relative;
}

.background-image > div:nth-child(1){
  background-color: rgb(0, 0, 0, 0.3);
  height: 100%;
  width: 100%;
  position: absolute;
}

.initial-recruitment {
  z-index: 1;
}

.garrison-buttons{
  z-index: 2;
}

.chooser {
  height: 72px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 16px;
}

.modal-body {
  border: 0px !important;
}

.switch {
  height: 55px;
  padding: 0 20px;
  font-size: 22px;
  min-width: 15px;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  border-color: transparent;
  color: #9e8a57;
  border-radius: 100px;
  line-height: 24px;
  box-sizing: border-box;
  white-space: nowrap;
  text-align: center;
  border: 1px solid transparent;
  box-shadow: 0 2px 0 rgb(0 0 0 / 2%);
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.645, 0.045, 0.355, 1);
  user-select: none;
  touch-action: manipulation;
  background: linear-gradient(180deg, rgba(31, 31, 34, 1) 0%, rgba(24, 27, 30, 1) 5%, rgba(24, 38, 45, 1) 100%);
}

.switch_active {
  border: 2px solid rgb(37, 167, 219);
  color: #9e8a57;
  cursor: pointer;
  font-weight: bold;
  background: linear-gradient(180deg, rgba(31, 31, 34, 1) 0%, rgba(24, 27, 30, 1) 5%, rgba(24, 38, 45, 1) 100%);
}
.switch_active:hover,
.switch:hover {
  transform: scale(0.97);
}

.confirmation-title{
  text-align: center;
  font-family: Trajan;
}

.footer-btn{
  flex-direction: row;
  gap: 1em;
}

.footer-btn > button{
  width: 100%;
}

.burn-content{
  color: #fff;
  font-family: Roboto;
  font-size: 0.9em;
}

.spinning-circle{
  content: url('../assets/magic-defender.png');
  position: absolute;
  z-index: 1;
  width: 90%;
  height: auto;
  max-width: 500px;
  animation-name: spin;
  animation-duration: 30s;
  animation-iteration-count: infinite;
  animation-timing-function: linear;
}

@keyframes spin {
    from {
        transform:rotate(0deg);
    }
    to {
        transform:rotate(360deg);
    }
}

.soul-container{
  z-index: 2;
}

.magic-circle{
  background-position: center;
  background-size:contain;
  background-repeat: no-repeat;
  background-color: rgba(255, 255, 255, 0);
  height: auto;
  padding: 10em ;
  display: flex;
  justify-content: center;
  align-items: center;
}


.magic-circle > .soul-container{
  background-color: #0F0F0F;
  border: 1px solid #3c3c3c;
  height: 100%;
  width: 100%;
  max-width: 150px;
  border-radius: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
  padding:0.7em 0px;
  flex-direction: column;
}


.magic-circle >.soul-container > p{
  font-family: Roboto;
  font-size: 1em;
  color: #fff;
  margin-bottom: 0px;
}

.soul-img{
  content: url('../assets/soul-icon.png');
  width: 50px;
  height: auto;
  -webkit-filter: drop-shadow(0px 0px 7px rgba(255, 255, 255, 0.521));
}

.glowUp{
  -webkit-filter: drop-shadow(0px 0px 15px rgb(255, 255, 255));
  transition: all 0.3s ease;
}


.recruit-btn{
  display: flex;
  flex-direction: column;
  height: 100%;
  margin-right: 15px;
  align-items: center;
  vertical-align: middle;
  justify-content: center;
  background-image: url('../assets/buttonOutline.svg');
  background-color: transparent;
  background-repeat: no-repeat;
  background-size: 100% 100%;
  object-fit: fill;
  padding: 10px 40px 10px 40px;
  border: none;
  font-family: Oswald;
  color: #fff;
  font-size: 17px;
  margin: auto;
  margin-right: -10px;
  cursor: pointer;
}

.btn-container{
  padding-right: 2em;
  margin-top: 3em;
}

.btn-container > div{
  text-align: center;
}

.btn-container > div > span > p {
  margin: 0px;
  font-family: Oswald;
  color: #fff;
  font-size: 1.3em;
}

.btn-container > div > span{
  margin: 0px;
  font-family: Roboto;
  color: #F0E2B6;
  font-size: 1;
}



.navbar-staking {
  display: flex;
  border: 1px solid #3c3c3c;
  box-sizing: border-box;
  border-radius: 100px;
  width: 100%;
}

.char-content{
   div.menu-nav{
    height: 60px;
    padding-left: 50px;
    padding-right: 50px;
    padding-top: 10px;
    padding-bottom: 10px;
    border-bottom: 1px solid #424A59;
    background-color:#000E29;
   }
 }

.char-info{
  padding: 50px 40px;
}

.garrisson-content > h1{
  font-family: Trajan;
  color: #9e8a57;
  margin-top: -20px;
  font-size: 2em;
  margin-bottom: 1em;
}


@media (max-width: 600px) {
  .char-content{
    padding: 0px;
  }
  .char-info{
    padding: 50px 40px;
  }
  .magic-circle{
    padding: 1em;
    width: 100%;
    margin-top: 2em;
  }

  .garrisson-content > h1{
    font-family: Trajan;
    color: #9e8a57;
    font-size: 1.5em;
    margin-top: -50px;
    margin-bottom: 1em;
  }

  .slippage-checkbox{
    justify-content: center;
    margin-top: 10px;
  }

  .spinning-circle{
    width: 110%;
  }

  .magic-circle > .soul-container{
    width: 70%;
    .soul-img{
      width: 40px;
    }
  }

  .scroll > h5{
    font-family: Roboto;
    color: rgba(255, 255, 255, 0.521);
    font-size: 0.9em;
    text-align: center;
    font-weight: 400;
  }

  .scroll > div {
    transform: rotate(90deg);
    animation: upDown 1s infinite alternate-reverse;
    text-align: center;
    p{
      font-size: 2em;
    }
  }


  @keyframes upDown {
    from{
      margin-top: 0px;
    }
    to{
      margin-top: 20px;
    }
  }

  .btn-container{
    padding-right: 0px;
    margin-top: 5em;
    margin-bottom: 2em;
  }

  .btn-container > div {
    padding: 7px 20px 7px 20px;
    padding-right: 20px;
    span{
      font-size: 0.8em;
    }
  }

  .char-content{
    div.menu-nav{
      padding-left: 0px;
      padding-right: 0px;
      padding-top: 10px;
      padding-bottom: 10px;
      border-bottom: 1px solid #424A59;
      background-color:#000E29;
    }
  }
}
</style>
