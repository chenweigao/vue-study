<template>
    <div>
        <el-form
            :inline="true"
            :model="formInline"
            class="demo-form-inline"
        >
            <el-form-item label="array_prepare">
                <el-input
                    v-model="formInline.card"
                    placeholder="array id"
                ></el-input>
            </el-form-item>

            <el-form-item label="channel parameter">
                <el-input
                    v-model="formInline.cardString"
                    placeholder="parameter"
                    value="jskjasdjk"
                ></el-input>
            </el-form-item>

            <el-form-item>
                <el-button
                    type="primary"
                    @click="onSubmit"
                >Add</el-button>
            </el-form-item>
        </el-form>

        <el-card class="box-card">
            Card Parameter:
            <h3>{{ nicInfo.parameter }}</h3>
            Card Information List:
            <h3></h3>

            <h3
                v-for="nic in nicInfo.ids"
                :key="nic.index"
                class="text item"
            >
                {{ nic }}
                <!-- <el-button icon="el-icon-delete" circle></el-button> -->
                <i
                    class="el-icon-delete"
                    @click="deleteCard(nic.index)"
                    style="float:right"
                ></i>
            </h3>
        </el-card>
        <el-card class="box-card">
            <el-form
                ref="form"
                :model="form"
                label-width="80px"
            >

                <el-form-item>

                    <el-select
                        v-model="selectedCard"
                        clearable
                        placeholder="Select your card"
                    >
                        <el-option
                            v-for="item in nicInfo.ids"
                            :key="item.index"
                            :label="item"
                            :value="item"
                        >
                        </el-option>
                    </el-select>
                    <el-select
                        v-model="selectedMode"
                        placeholder="Select your mode"
                    >
                        <el-option
                            v-for="item in modes"
                            :key="item.index"
                            :label="item"
                            :value="item"
                        >
                        </el-option>
                    </el-select>
                    <el-button
                        @click="configCard"
                        type="success"
                    >Config</el-button>
                    <!-- <el-button @click="fetchState">Send</el-button> -->
                </el-form-item>

                <el-form-item label="freq">
                    <el-input v-model="form.freq"></el-input>
                </el-form-item>

                <el-form-item
                    label="txcm"
                    v-if="selectedMode !='logger'"
                >
                    <el-input-number
                        v-model="form.txcm"
                        :min="1"
                        :max="7"
                        label="txcm"
                    ></el-input-number>
                </el-form-item>
                <el-form-item label="rxcm">
                    <el-input-number
                        v-model="form.rxcm"
                        :min="1"
                        :max="7"
                        label="rxcm"
                    ></el-input-number>
                </el-form-item>
                <el-form-item
                    label="ness"
                    v-if="selectedMode !='logger'"
                >
                    <el-input-number
                        v-model="form.ness"
                        :min="0"
                        :max="2"
                        label="rxcm"
                    ></el-input-number>
                </el-form-item>

                <el-form-item label="pll">
                    <el-input v-model="form.pll"></el-input>
                </el-form-item>

                <el-form-item
                    label="Policy"
                    v-if="selectedMode !='logger'"
                >
                    <el-select
                        v-model="form.region"
                        placeholder="Policy"
                    >
                        <el-option
                            label="chansel"
                            value="chansel"
                        ></el-option>
                        <el-option
                            label="fastcc"
                            value="fastcc"
                        ></el-option>
                        <el-option
                            label="reset"
                            value="reset"
                        ></el-option>
                        <el-option
                            label="default"
                            value="default"
                        ></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item
                    label="repeat"
                    v-if="selectedMode =='injector' || selectedMode == 'initiator'"
                >
                    <el-input v-model="form.repeat"></el-input>
                </el-form-item>
                <el-form-item
                    label="delay"
                    v-if="selectedMode =='injector' || selectedMode == 'initiator'"
                >
                    <el-input v-model="form.delay"></el-input>
                </el-form-item>
                <el-form-item
                    label="cf"
                    v-if="selectedMode =='injector' || selectedMode == 'initiator'"
                >
                    <el-input v-model="form.cf"></el-input>
                </el-form-item>
                <el-form-item
                    label="sf"
                    v-if="selectedMode =='injector' || selectedMode == 'initiator'"
                >
                    <el-input v-model="form.sf"></el-input>
                </el-form-item>
                <el-form-item
                    label="delayed start"
                    v-if="selectedMode =='injector' || selectedMode == 'initiator'"
                >
                    <el-input v-model="form.delayedStart"></el-input>
                </el-form-item>

            </el-form>
        </el-card>
        <el-card class="box-card">
            <div
                slot="header"
                class="clearfix"
            >
                <span>Config Json:</span>
                <el-button
                    style="float: right; padding: 3px 0"
                    type="text"
                    @click="fetchState"
                >Send</el-button>
            </div>
            <div>
                {{ configResult }}
            </div>
        </el-card>

    </div>
</template>
<script>
var dgram = require('dgram')
var udpClient = dgram.createSocket('udp4')
export default {
  data () {
    return {
      formInline: {
        card: '',
        cardString: ''
      },
      nicInfo: {
        ids: [],
        parameter: ''
      },
      modes: ['logger', 'injector', 'responder', 'initiator'],
      selectedCard: '',
      selectedMode: 'responder',
      selectedPolicy: '',
      selectedNicID: '',
      form: {
        NICId: '',
        freq: '',
        txcm: 7,
        rxcm: 7,
        ness: 0,
        pll: '',
        policy: '',
        repeat: '',
        delay: '',
        cf: '',
        sf: '',
        delayedStart: '',
        mode: ''
      },
      configResult: []
    }
  },
  methods: {
    onSubmit () {
      this.nicInfo.ids.push(this.formInline.card)
      this.nicInfo.parameter = this.formInline.cardString
      console.log(this.nicInfo)
    },
    deleteCard (index) {
      this.nicInfo.ids.splice(index, 1)
      console.log(this.nicInfo.ids)
    },
    configCard () {
      this.form.mode = this.selectedMode
      this.form.NICId = this.formInline.card
      for (var key in this.form) {
        if (this.form[key] === '') {
          delete this.form[key]
        }
      }
      this.configResult.push(this.form)
      console.log(this.configResult)
      //   var SendBuff = JSON.stringify(this.configResult)
      //   var SendLen = SendBuff.length
      //   udpClient.send(SendBuff, 0, SendLen, 7788, '127.0.0.1')
    },
    fetchState () {
      var SendBuff = JSON.stringify(this.configResult)
      var SendLen = SendBuff.length
      udpClient.send(SendBuff, 0, SendLen, 7788, '127.0.0.1')
    }

  },
  created () {
    var self = this
    udpClient.on('message', function (msg, rinfo) {
      try {
        var data = JSON.parse(msg)
        if (data) {
          self.processList = data
        }
      } catch (e) { }
      console.log(
        `receive message from ${rinfo.address}:${rinfo.port}：${msg}`
      )
    })
    // window.setInterval(self.fetchState, 1000)
  }
}
</script>