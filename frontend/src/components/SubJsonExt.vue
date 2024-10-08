<template>
  <v-card>
    <v-row>
      <v-col cols="12" sm="6" md="3">
        <v-select
          v-model="ruleToDirect"
          :items="geoList"
          :label="$t('setting.toDirect')"
          multiple
          chips
          hide-details
        ></v-select>
      </v-col>
      <v-col cols="12" sm="6" md="3">
        <v-select
          v-model="ruleToBlock"
          :items="geoList"
          :label="$t('setting.toBlock')"
          multiple
          chips
          hide-details
        ></v-select>
      </v-col>
    </v-row>
    <v-row  v-if="enableLog">
      <v-col cols="12" sm="6" md="3" lg="2">
        <v-select
          hide-details
          :label="$t('basic.log.level')"
          :items="levels"
          v-model="subJsonExt.log.level">
        </v-select>
      </v-col>
      <v-col cols="12" sm="6" md="3" lg="2">
        <v-switch v-model="subJsonExt.log.timestamp" color="primary" :label="$t('setting.timestamp')" hide-details />
      </v-col>
    </v-row>
    <v-row v-if="enableDns">
      <v-col cols="12" sm="6" md="3" lg="2">
        <v-text-field
          v-model="proxyDns"
          hide-details
          :label="$t('setting.globalDns')"
        ></v-text-field>
      </v-col>
      <v-col cols="12" sm="6" md="3" lg="2">
        <v-text-field
          v-model="directDns"
          hide-details
          clearable
          :label="$t('setting.directDns')"
        ></v-text-field>
      </v-col>
      <v-col cols="12" sm="6" md="3" v-if="directDns.length>0">
        <v-select
          v-model="dnsToDirect"
          :items="geositeList"
          :label="$t('setting.toDirectDns')"
          multiple
          chips
          hide-details
        ></v-select>
      </v-col>
    </v-row>
    <v-card-actions>
      <v-spacer></v-spacer>
      <v-menu v-model="menu" :close-on-content-click="false" location="start">
        <template v-slot:activator="{ props }">
          <v-btn v-bind="props" hide-details variant="tonal">{{ $t('setting.jsonSubOptions') }}</v-btn>
        </template>
        <v-card>
          <v-list>
            <v-list-item>
              <v-switch v-model="enableLog" color="primary" :label="$t('basic.log.title')" hide-details></v-switch>
            </v-list-item>
            <v-list-item>
              <v-switch v-model="enableDns" color="primary" label="DNS" hide-details></v-switch>
            </v-list-item>
            <v-list-item>
              <v-switch v-model="enableExp" color="primary" label="Experimental" hide-details></v-switch>
            </v-list-item>
          </v-list>
        </v-card>
      </v-menu>
    </v-card-actions>
  </v-card>
</template>

<script lang="ts">
export default {
  props: ['settings'],
  data() {
    return {
      menu: false,
      subJsonExt: <any>{},
      levels: ["trace", "debug", "info", "warn", "error", "fatal", "panic"],
      defaultLog: {
        "level": "info",
        "timestamp": true
      },
      defaultExp: {
        "clash_api": {
          "external_controller": "127.0.0.1:9090",
          "external_ui": "ui",
          "secret": "",
          "external_ui_download_url": "https://mirror.ghproxy.com/https://github.com/MetaCubeX/Yacd-meta/archive/gh-pages.zip",
          "external_ui_download_detour": "direct",
          "default_mode": "rule"
        },
        "cache_file": {
          "enabled": true,
          "store_fakeip": false
        }
      },
      defaultDns: {
        "servers": [
          {
            "address": "tcp://8.8.8.8",
            "detour": "proxy",
            "address_resolver": "local-dns",
            "tag": "proxy-dns"
          },
          {
            "tag": "local-dns",
            "address": "local",
            "detour": "direct"
          },
          {
            "address": "rcode://success",
            "tag": "block"
          }
        ],
        "rules": [
          {
            "clash_mode": "Global",
            "source_ip_cidr": [
              "172.19.0.0/30"
            ],
            "server": "proxy-dns"
          },
          {
            "source_ip_cidr": [
              "172.19.0.0/30"
            ],
            "server": "proxy-dns"
          }
        ],
        "final": "local-dns",
        "strategy": "prefer_ipv4"
      },
      geositeList: [
        { title: "RU Private", value: "ru-geosite" },
      ],
      geoList: [
        { title: "RU Site", value: "ru-geosite" },
        { title: "Site-Private", value: "geosite-private" },
        { title: "RU IP ", value: "ru-geoip" },
        { title: "Telegram", value: "geosite-telegram" },
        { title: "WhatsApp", value: "geosite-whatsapp" },
        { title: "RU Android apps", value: "ru-android-apps" },
        { title: "RU vk.com", value: "ru-geosite-vk" },
      ],
      geo: [
        {
          tag: "ru-geosite",
          type: "remote",
          format: "binary",
          url: "https://raw.githubusercontent.com/jeffscrum/sb-rulesets/master/custom/ru-sites.srs",
          download_detour: "direct"
        },
        {
          tag: "geosite-private",
          type: "remote",
          format: "binary",
          url: "https://raw.githubusercontent.com/jeffscrum/sb-rulesets/master/custom/private.srs",
          download_detour: "direct"
        },
        {
          tag: "ru-geoip",
          type: "remote",
          format: "binary",
          url: "https://testingcf.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@sing/geo/geoip/ru.srs",
          download_detour: "direct"
        },
        {
          tag: "geosite-telegram",
          type: "remote",
          format: "binary",
          url: "https://raw.githubusercontent.com/jeffscrum/sb-rulesets/master/custom/telegram.srs",
          download_detour: "direct"
        },
        {
          tag: "geosite-whatsapp",
          type: "remote",
          format: "binary",
          url: "https://raw.githubusercontent.com/jeffscrum/sb-rulesets/master/custom/whatsapp.srs",
          download_detour: "direct"
        },
        {
          tag: "ru-android-apps",
          type: "remote",
          format: "binary",
          url: "https://raw.githubusercontent.com/jeffscrum/sb-rulesets/master/custom/ru-android-apps.srs",
          download_detour: "direct"
        },
        {
          tag: "ru-geosite-vk",
          type: "remote",
          format: "binary",
          url: "https://raw.githubusercontent.com/jeffscrum/sb-rulesets/master/custom/vk.srs",
          download_detour: "direct"
        },
      ],
    }
  },
  computed: {
    enableLog: {
      get() :boolean { return this.subJsonExt?.log != undefined },
      set(v:boolean) { v ? this.subJsonExt.log = this.defaultLog : delete this.subJsonExt.log }
    },
    enableDns: {
      get() :boolean { return this.subJsonExt?.dns != undefined },
      set(v:boolean) {
        if (v) {
          this.subJsonExt.dns = this.defaultDns
          if (this.rules == undefined) this.subJsonExt.rules = []
          this.subJsonExt.rules.unshift({ protocol: "dns", outbound: "dns-out" })
        } else {
          delete this.subJsonExt.dns
          const ruleDnsIndex = this.subJsonExt?.rules?.findIndex((r:any) => r.protocol = "dns" && r.outbound == "dns-out")
          if (ruleDnsIndex >= 0) this.subJsonExt.rules.splice(ruleDnsIndex,1)
          if (this.rules.length == 0) delete this.subJsonExt.rules
        }
      }
    },
    enableExp: {
      get() :boolean { return this.subJsonExt?.experimental != undefined },
      set(v:boolean) { v ? this.subJsonExt.experimental = this.defaultExp : delete this.subJsonExt.experimental }
    },
    dns():any { return this.subJsonExt?.dns?? undefined },
    proxyDns: {
      get() :string { return this.dns?.servers[0]?.address?? "" },
      set(v:string) { this.dns.servers[0].address = v.length>0 ? v : "8.8.8.8" }
    },
    directDns: {
      get() :string { return this.dns?.servers?.findLast((d:any) => d.tag == "direct-dns")?.address?? "" },
      set(v:string) {
        const sIndex = this.dns.servers.findIndex((d:any) => d.tag == "direct-dns")
        if (v?.length>0) {
          if (sIndex === -1) {
            this.dns.servers.push({ tag: "direct-dns", address: v, detour: "direct" })
            this.dns.rules.push({ clash_mode: "Direct", server: "direct-dns" })
          } else {
            this.dns.servers[sIndex].address = v
          }
        } else {
          this.dns.servers.splice(sIndex,1)
          this.dns.rules = this.dns.rules.filter((r:any) => r.server != "direct-dns")
        }
      },
    },
    dnsToDirect: {
      get() :string[] {
        const ruleIndex = this.dns?.rules?.findIndex((r:any) => r.server == "direct-dns" && Object.hasOwn(r,'rule_set'))
        return ruleIndex >= 0 ? this.dns.rules[ruleIndex].rule_set : []
      },
      set(v:string[]) {
        const ruleIndex = this.dns?.rules?.findIndex((r:any) => r.server == "direct-dns" && Object.hasOwn(r,'rule_set'))
        if (v.length>0) {
          if (ruleIndex >= 0){
            this.dns.rules[ruleIndex].rule_set = v
          } else {
            this.dns.rules.push({ rule_set: v, server: "direct-dns" })
          }
        } else {
          if (ruleIndex != -1) this.dns.rules.splice(ruleIndex,1)
        }
        this.updateRuleSets()
      }
    },
    rules():any { return this.subJsonExt?.rules?? undefined },
    ruleToDirect: {
      get() :string[] {
        const ruleIndex = this.rules?.findIndex((r:any) => r.outbound == "direct" && Object.hasOwn(r,'rule_set'))
        return ruleIndex >= 0 ? this.rules[ruleIndex].rule_set : []
      },
      set(v:string[]) {
        const ruleIndex = this.rules?.findIndex((r:any) => r.outbound == "direct" && Object.hasOwn(r,'rule_set'))
        if (v.length>0) {
          if (ruleIndex >= 0){
            this.rules[ruleIndex].rule_set = v
          } else {
            if (this.rules == undefined) this.subJsonExt.rules = []
            this.rules.push({ rule_set: v, outbound: "direct" })
          }
        } else {
          if (ruleIndex != -1) this.rules.splice(ruleIndex,1)
        }
        this.updateRuleSets()
      }
    },
    ruleToBlock: {
      get() :string[] {
        const ruleIndex = this.rules?.findIndex((r:any) => r.outbound == "block" && Object.hasOwn(r,'rule_set'))
        return ruleIndex >= 0 ? this.rules[ruleIndex].rule_set : []
      },
      set(v:string[]) {
        const ruleIndex = this.rules?.findIndex((r:any) => r.outbound == "block" && Object.hasOwn(r,'rule_set'))
        if (v.length>0) {
          if (ruleIndex >= 0){
            this.rules[ruleIndex].rule_set = v
          } else {
            if (this.rules == undefined) this.subJsonExt.rules = []
            this.rules.push({ rule_set: v, outbound: "block" })
          }
        } else {
          if (ruleIndex != -1) this.rules.splice(ruleIndex,1)
        }
        this.updateRuleSets()
      }
    }
  },
  methods: {
    updateRuleSets(){
      let tags = <string[]>[]
      if (this.dns?.rules?.length>0) this.dns.rules.forEach((r:any) => { if (r.rule_set) tags.push(...r.rule_set) })
      if (this.rules?.length>0) this.rules.forEach((r:any) => { if (r.rule_set) tags.push(...r.rule_set) })
      if (tags.length>0){
        this.subJsonExt.rule_set = this.geo.filter((g:any) => tags.includes(g.tag))
      } else {
        delete this.subJsonExt.rule_set
      }
      if (this.rules.length == 0) delete this.subJsonExt.rules
    }
  },
  mounted(){
    this.subJsonExt = this.$props.settings?.subJsonExt?.length>0 ? JSON.parse(this.$props.settings.subJsonExt) : <any>{}
  },
  watch:{
    subJsonExt:{
      handler(v) {
        this.$props.settings.subJsonExt = Object.keys(v).length>0 ? JSON.stringify(v, null, 2) : ""
      },
      deep: true
    },
  }
}
</script>
