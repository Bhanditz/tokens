<template>
  <section class="token-about">
    <header>About {{token.toUpperCase()}}</header> 
    <ul class="about-list">
      <li>
        <label>Publisher</label>
        <span>{{about.owner}}</span>
      </li>
      <li>
        <label>Publish Time</label>
        <span>{{timeFormat(about.start_time)}}</span>
      </li>
      <li>
        <label>Total Amount</label>
        <span>{{((+about.base_stake + Number(about.base_option) - Number(about.deserved_option)) / Math.pow(10, decimals)).toFixed(decimals)}}</span>
      </li>
      <li>
        <label>Base Option</label>
        <span>{{(about.base_option / 10000).toFixed(decimals)}}</span>
      </li>
      <li>
        <label>EOS Pool</label>
        <span>{{about.eosPool}}</span>
      </li>
      <li>
        <label>Buy Fee Rate</label>
        <span>{{referFeePercent}}%</span>
      </li>
      <li>
        <label>Sell Fee Rate</label>
        <span>{{about.feePercent}}%</span>
      </li>
      <li>
        <label>Lockup</label>
        <span>{{rangeFormat(about.lock_up_period)}}</span>
      </li>
    </ul> 
    <ul 
      v-if="social"
      class="social-list">
      <li @click="openTab(social.website)">
        <el-tooltip
          effect="dark" 
          content="Website" 
          placement="top-start">
          <font-awesome-icon icon="home" />
        </el-tooltip>
      </li>
      <li @click="openTab(social.community)">
        <el-tooltip
          effect="dark" 
          content="Community" 
          placement="top-start">
          <font-awesome-icon icon="comments" />
        </el-tooltip>
      </li>
      <li @click="openTab(social.social)">
        <font-awesome-icon :icon="['fab', 'twitter']" />
      </li>
      <li @click="openTab(social.medium)">
        <font-awesome-icon :icon="['fab', 'medium-m']" />
      </li>
      <li @click="openTab(social.github)">
        <font-awesome-icon :icon="['fab',  'github']" />
      </li>
    </ul>
  </section> 
</template>

<script>
import fetch from '@/utils/api';
import api from '@/utils/eos';
import { feePercent, hexTransform } from '@/utils/math';
import logoPub from '@/assets/pub.png';

export default {
  mounted() {
    this.fetchSocial();
    this.getBalance();
    this.fetchToken();
    this.fetchReferFee();
  },

  data() {
    return {
      decimals: 0,
      logoPub,
      about: {},
      social: {}
    };
  },

  watch: {
    token() {
      this.fetchToken();
      this.fetchSocial();
      this.getBalance();
      this.fetchReferFee();
    },

    account() {
      this.getBalance();
    }
  },

  computed: {
    token() {
      return this.$store.state.token;
    },
    account() {
      return this.$store.state.account; 
    }
  },

  methods: {
    openTab(url) {
      window.open(url);
    },

    fetchToken() {
      api.getTableRows({
        json: true,
        code: 'tokendapppub',
        scope: this.token.toUpperCase(),
        table: 'games'
      }).then(({ rows }) => {
        this.about = rows[0];
        this.about.feePercent = feePercent(this.about);
        this.about.eosPool = (hexTransform(this.about.eos) - hexTransform(this.about.base_eos)).toFixed(4);
      }); 
    },

    fetchReferFee() {
      api.getTableRows({
        json: true,
        code: 'tokendapppub',
        scope: this.token.toUpperCase(),
        table: 'refer'
      }).then(({ rows }) => {
        if (rows.length == 1) {
          this.referFeePercent = rows[0].fee_percent/100;
        } else {
          this.referFeePercent = 0;
        }  
      }); 
    },

    fetchSocial() {
      fetch(`token/detail?name=${this.token}`).then(({ token }) => {
        this.social = token; 
      }).catch(() => {
        this.social = undefined; 
      });
    },

    getBalance() {
      api.getTableRows({
        json: true,
        code: 'tokendapppub',
        table: 'stat',
        scope: this.token.toUpperCase()
      }).then(({ rows }) => {
        const { max_supply } = rows[0]; 
        this.decimals = (max_supply.match(/[\d\.]+/)[0].split('.')[1] || '').length;
      });
    },

    timeFormat(raw) {
      const date = new Date(raw * 1000);
      const year = date.getFullYear();
      const month = date.getMonth() + 1;
      const day = date.getDate();
      return `${year}-${month}-${day}`;
    },

    rangeFormat(raw) {
      if (Number(raw) === 0) return 0; 
      let time = raw * 1000;
      let year = (time / (864E5 * 365)).toFixed(0);
      let month = ((time - year * 864E5 * 365) / 864E5 * 30).toFixed(0);
      let day = ((time - year * 864E5 * 365 - month * 864E5 * 30) / 864E5).toFixed(0);
      if (year <= 0 && month <= 0 && day <= 0) return `less than 1 day`;
      return `${year > 0 ? year + ' years ' : ''}${month > 0 ? month + ' months ' : ''}${day > 0 ? day + ' days ' : ''}`;
    }
  }
};
</script>

<style scoped>
.token-about {
  background-color: #fff;
  box-shadow: rgba(114, 115, 119, 0.05) 0px 4px 14px;
  border: 1px solid #DBE1E8;
  border-radius: 6px;
  padding: 24px 32px 16px;
  margin-right: 30px;
}

.token-about > header {
  font-weight: 600;
  color: rgb(80, 92, 108);
  margin-bottom: 30px;
  line-height: 1.5;
}

.about-list > li {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 15px;
}

.about-list > li > label {
  color: #2968C9;
  margin-right: 100px;
}

.about-list > li > span {
  color: rgb(78, 92, 110);
  line-height: 1.5;
  font-weight: 500;
}

.logo-pub {
  width: 45px;
  margin-bottom: 15px;
}

.social-list {
  display: flex;
  margin-top: 100px;
}

.social-list > li {
  margin-right: 30px;
}

.social-list > li {
  cursor: pointer; 
  opacity: .5;
  transition: opacity ease 300ms;
}

.social-list > li:hover {
  opacity: 1; 
}

.social-list > li > a {
  color: #3869C2; 
  text-decoration: none;
}

.social-list > li > a:hover {
  opacity: .5;
}
</style>

