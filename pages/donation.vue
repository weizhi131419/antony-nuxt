<template>
  <div id="comments">
    <div class="grid grid-centered">
      <div class="grid-cell" id="grid-cell">
        <article class="article reveal">
          <div id="load" class="donate-load">
            <h3 class="donate-title">{{ $t('lang.donation.line1') }}</h3>
            <button
              type="button"
              class="btn btn-primary btn-sm donate-switch"
              @click="switchLanguage()"
            >
              <i class="ri-earth-line"></i>
              {{ $t('lang.donation.line2') }}
            </button>
            <p class="page-p donate-p">{{ $t('lang.donation.line3') }}</p>
            <div style="display:flex">
              <div class="payment-div-1">
                <a
                  target="_blank"
                  href="https://static.ouorz.com/alipay.png"
                  style="text-decoration:none"
                >
                  <h1 class="donate-icon">
                    <i class="ri-alipay-line"></i>
                  </h1>
                  <h4 class="donate-h4">{{ $t('lang.donation.line4') }}</h4>
                </a>
              </div>
              <div class="payment-div-1-1">
                <a
                  target="_blank"
                  href="https://static.ouorz.com/wechatpay.png"
                  style="text-decoration:none"
                >
                  <h1 class="donate-icon">
                    <i class="ri-wechat-pay-line"></i>
                  </h1>
                  <h4 class="donate-h4">{{ $t('lang.donation.line5') }}</h4>
                </a>
              </div>
            </div>

            <div class="payment-div-2 payment-div-2-2" id="view-bitcoin">
              <a href="#" style="text-decoration:none">
                <h4 class="donate-bitcoin">
                  <i class="ri-bit-coin-line"></i>
                  {{ $t('lang.donation.line6') }}
                </h4>
              </a>
            </div>
            <b-tooltip
              target="view-bitcoin"
              triggers="hover"
              placement="top"
            >bc1qz2kgqp26wtel6n7rl0cw053pxgtwt5vrr5hyd7pqmmjfhqxex8dq8fknpx</b-tooltip>

            <div class="payment-div-2">
              <a
                target="_blank"
                href="https://paypal.me/helipeng?locale.x=zh_XC"
                style="text-decoration:none"
              >
                <h4 class="donate-paypal">
                  <i class="ri-paypal-line"></i>
                  {{ $t('lang.donation.line7') }}
                </h4>
              </a>
            </div>

            <p class="page-p donate-p">
              {{ $t('lang.donation.line8.line1') }}
              <a
                href="https://www.ouorz.com/post/126#in10"
                target="_blank"
              >{{ $t('lang.donation.line8.line2') }}</a>
              {{ $t('lang.donation.line8.line3') }}
              <br/><br/>
              {{ $t('lang.donation.line13') }}
              <a href="https://www.blockchain.com/api/exchange_rates_api" target="_blank">BlockChain</a>
              {{ $t('lang.donation.line14') }}
            </p>
            <table border="1" class="donate-table">
              <tbody>
                <tr>
                  <template>
                    <th>{{ $t('lang.donation.line9') }}</th>
                    <th>
                      {{ $t('lang.donation.line10') }}
                      <b-button
                        style="margin-top: -4.5px;padding: 0px 7px;"
                        size="sm"
                        @click="switchCurrency()"
                      >{{ $t('lang.donation.line12') }}</b-button>
                    </th>
                    <th>{{ $t('lang.donation.line11') }}</th>
                  </template>
                </tr>
                <tr v-for="(donor,index) in donorsCurrent" :key="index">
                  <td>{{ donor.name }}</td>
                  <td>{{ donor.unit }}{{ toFixorNot(donor.amount) }}</td>
                  <td>{{ donor.date }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </article>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Donation',
  async asyncData(context) {
    let res = await Promise.all([
      // 获取博客文章数据
      context.$axios
        .get('https://blog.ouorz.com/wp-content/themes/peg/com/data/donors.php')
        .then(response => {
          return response.data.donors
        })
    ])
    return {
      donorsCurrent: res[0],
      donorsCNY: res[0],
      donorsBTC: null
    }
  },
  data() {
    return {
      lang: this.$i18n.locale,
      donorsBTC: null,
      donorsCNY: null,
      donorsCurrent: null,
      unit: 'cny'
    }
  },
  head() {
    return {
      title: 'TonyHe - 赞助与列表',
      meta: [
        {
          hid: 'description',
          name: 'description',
          content: 'TonyHe 的赞助与赞助列表'
        },
        {
          hid: 'keywords',
          name: 'keywords',
          content:
            '贺莉朋,Tony,TonyHe,博客,个人博客,Personal Blog,blog,tony,tonyhe,赞助,赞助列表,打赏,喂食'
        }
      ]
    }
  },
  methods: {
    switchLanguage: function() {
      if (this.lang === 'zh-CN') {
        this.lang = 'en-US'
        this.$i18n.locale = this.lang
      } else {
        this.lang = 'zh-CN'
        this.$i18n.locale = this.lang
      }
    },
    cnyTobtc: async function(value) {
      let res = await Promise.all([
        // 获取博客文章数据
        this.$axios
          .get(
            'https://blockchain.info/tobtc?cors=true&currency=CNY&value=' +
              value
          )
          .then(response => {
            return response.data
          })
      ])
      return res[0]
    },
    switchCurrency: function() {
      if (this.unit == 'btc') {
        this.unit = 'cny'
        this.$axios
          .get(
            'https://blog.ouorz.com/wp-content/themes/peg/com/data/donors.php'
          )
          .then(response => {
            this.donorsCNY = response.data.donors
            this.donorsCurrent = this.donorsCNY
          })
      } else {
        this.unit = 'btc'
        this.donorsBTC = this.donorsCurrent
        for (let i = 0; i < this.donorsBTC.length; i++) {
          if (this.donorsBTC[i].unit !== '฿') {
            this.cnyTobtc(parseFloat(this.donorsBTC[i].amount)).then(value => {
              this.donorsBTC[i].amount = value.toFixed(7)
            })
            this.donorsBTC[i].unit = '฿'
          } else {
            this.donorsBTC[i].amount = this.donorsBTC[i].amount.toFixed(7)
          }
        }
        this.donorsCurrent = this.donorsBTC
      }
    },
    toFixorNot: function(value) {
      if (this.unit == 'btc') {
        return value
      } else {
        return parseFloat(value).toFixed(2)
      }
    }
  }
}
</script>