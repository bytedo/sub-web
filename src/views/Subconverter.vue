```
<template>
  <div>
    <el-row style="margin-top: 10px">
      <el-col>
        <!-- Page Header -->
        <div class="page-header">
           <div class="header-left">
              <i class="el-icon-s-promotion header-icon" style="font-size: 32px; margin-right: 15px; color: #409EFF;"></i>
              <div class="header-text">
                 <div class="header-title">Sub Converter</div>
                 <div class="header-subtitle">订阅链接转换工具</div>
              </div>
           </div>
           <div class="header-right">{{ currentTime }}</div>
        </div>

        <div class="common-card main-config-card">
          <div class="main-container">
            <!-- Form moved inside here essentially, but keeping structure -->
            <el-form :model="form" label-width="140px" label-position="left" style="width: 100%">
          
          <div class="form-section">
            <el-form-item label="模式设置">
               <el-radio-group v-model="advanced" size="small" fill="#2ed573">
                  <el-radio-button label="1">基础模式</el-radio-button>
                  <el-radio-button label="2">进阶模式</el-radio-button>
               </el-radio-group>
            </el-form-item>
            
            <el-form-item label="订阅链接">
              <el-input
                v-model="form.sourceSubUrl"
                type="textarea"
                rows="3"
                placeholder="支持订阅或ss/ssr/vmess链接，多个链接每行一个或用 | 分隔"
                @blur="saveSubUrl"
              />
            </el-form-item>

            <el-form-item label="客户端">
               <el-select v-model="form.clientType" style="width: 100%" placeholder="请选择客户端">
                  <el-option v-for="(v, k) in options.clientTypes" :key="k" :label="k" :value="v"></el-option>
               </el-select>
            </el-form-item>

            <!-- Advanced Configuration (Hidden by default) -->
            <transition name="el-zoom-in-top">
              <div v-if="advanced === '2'" class="advanced-section">
                
                <el-row :gutter="20">
                    <el-col :span="12">
                      <el-form-item label="后端地址">
                        <el-autocomplete
                          style="width: 100%"
                          v-model="form.customBackend"
                          :fetch-suggestions="backendSearch"
                          placeholder="建议自行搭建后端服务"
                        ></el-autocomplete>
                      </el-form-item>
                    </el-col>
                    <el-col :span="12">
                      <el-form-item label="远程配置">
                        <el-select
                          v-model="form.remoteConfig"
                          allow-create
                          filterable
                          placeholder="请选择"
                          style="width: 100%"
                        >
                          <el-option-group
                            v-for="group in options.remoteConfig"
                            :key="group.label"
                            :label="group.label"
                          >
                            <el-option
                              v-for="item in group.options"
                              :key="item.value"
                              :label="item.label"
                              :value="item.value"
                            ></el-option>
                          </el-option-group>
                        </el-select>
                      </el-form-item>
                    </el-col>
                </el-row>

                 <el-row :gutter="20">
                    <el-col :span="8">
                      <el-form-item label="Include">
                        <el-input v-model="form.includeRemarks" placeholder="关键字(正则)" />
                      </el-form-item>
                    </el-col>
                    <el-col :span="8">
                       <el-form-item label="Exclude">
                        <el-input v-model="form.excludeRemarks" placeholder="关键字(正则)" />
                      </el-form-item>
                    </el-col>
                     <el-col :span="8">
                       <el-form-item label="FileName">
                          <el-input v-model="form.filename" placeholder="文件名" />
                       </el-form-item>
                     </el-col>
                 </el-row>

                 <!-- Custom Params Display -->
                 <div v-if="customParams.length > 0" class="custom-params-section">
                    <div class="section-title-small">自定义参数</div>
                    <div v-for="(param, i) in customParams" :key="i" class="custom-param-row">
                       <el-input v-model="param.name" placeholder="参数名" size="medium" style="width: 45%; margin-right: 10px;"></el-input>
                       <span style="font-weight: bold; margin-right: 10px;">:</span>
                       <el-input v-model="param.value" placeholder="参数值" size="medium" style="width: 45%; margin-right: 10px;"></el-input>
                       <el-button type="text" icon="el-icon-close" style="color: #f56c6c; font-size: 16px;" @click="customParams.splice(i, 1)"></el-button>
                    </div>
                 </div>

                 <div class="divider"></div>

                 <!-- Options Grid (Inside Main Card) -->
                 <div class="options-area-internal">
                    <el-row :gutter="20">
                      
                      <!-- Column 1: General Options -->
                      <el-col :xs="24" :sm="12" :md="8" :lg="8">
                        <div class="common-card option-card internal-card">
                          <div class="card-header">通用选项</div>
                          <div class="card-content vertical-checkboxes">
                             <el-checkbox v-model="form.emoji" label="Emoji"></el-checkbox>
                             <el-checkbox v-model="form.nodeList" label="输出为 Node List"></el-checkbox>
                             <el-checkbox v-model="form.sort" label="排序节点"></el-checkbox>
                             <el-checkbox v-model="form.appendType" label="节点类型"></el-checkbox>
                          </div>
                        </div>
                      </el-col>

                      <!-- Column 2: Filter & Rules -->
                      <el-col :xs="24" :sm="12" :md="8" :lg="8">
                        <div class="common-card option-card internal-card">
                           <div class="card-header">过滤与规则</div>
                           <div class="card-content vertical-checkboxes">
                              <el-checkbox v-model="form.scv" label="跳过证书验证"></el-checkbox>
                              <el-checkbox v-model="form.udp" @change="needUdp = true" label="启用 UDP"></el-checkbox>
                              <el-checkbox v-model="form.fdn" label="过滤非法节点"></el-checkbox>
                              <el-checkbox v-model="form.expand" label="规则展开"></el-checkbox>
                           </div>
                        </div>
                      </el-col>

                      <!-- Column 3: Custom & Special -->
                      <el-col :xs="24" :sm="12" :md="8" :lg="8">
                         <div class="common-card option-card internal-card">
                           <div class="card-header">定制功能</div>
                           <div class="card-content vertical-checkboxes">
                              <el-checkbox v-model="form.tpl.surge.doh" label="Surge.DoH"></el-checkbox>
                              <el-checkbox v-model="form.tpl.clash.doh" label="Clash.DoH"></el-checkbox>
                              <el-checkbox v-model="form.insert" label="网易云"></el-checkbox>
                              
                              <div class="custom-buttons">
                                 <el-button size="mini" icon="el-icon-plus" @click="addCustomParam">添加参数</el-button>
                              </div>
                           </div>
                         </div>
                      </el-col>
                    </el-row>
                 </div>
                 
              </div>
            </transition>
          </div>
        </el-form>
      </div>

      <!-- Action Area (Buttons) -->
      <div class="action-card common-card" style="margin-top: 20px; padding: 20px;">
         <el-form label-width="80px">
           <el-form-item label="定制订阅">
              <el-input v-model="customSubUrl" readonly size="medium">
                 <el-button slot="append" v-clipboard:copy="customSubUrl" v-clipboard:success="onCopy" icon="el-icon-document-copy">复制</el-button>
              </el-input>
           </el-form-item>
         </el-form>
         <div style="text-align: center; margin-top: 20px;">
            <el-button
              type="danger"
              class="generate-btn"
              @click="makeUrl"
              :disabled="form.sourceSubUrl.length === 0"
              icon="el-icon-magic-stick"
              round
            >
              生成订阅链接
            </el-button>
         </div>
      </div>

      </div>
    </el-col>
    </el-row>
  </div>
</template>

<script>
const defaultBackend = process.env.VUE_APP_SUBCONVERTER_DEFAULT_BACKEND + '/sub?'

export default {
  data() {
    return {
      backendVersion: "",
      advanced: "2",
      isPC: true,
      options: {
        clientTypes: {
          Clash: "clash",
          Surge: "surge&ver=4",
          Quantumult: "quan",
          QuantumultX: "quanx",
          Mellow: "mellow",
          Surfboard: "surfboard",
          Loon: "loon",
          singbox: "singbox",
          ss: "ss",
          ssd: "ssd",
          sssub: "sssub",
          ssr: "ssr",
          ClashR: "clashr",          
          V2Ray: "v2ray",
          Trojan: "trojan",
          Surge3: "surge&ver=3",
        },
        backendOptions: [{ value: "http://127.0.0.1:25500/sub?" }],
        remoteConfig: [
          {
            label: "universal",
            options: [
              {
                label: "No-Urltest",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/universal/no-urltest.ini"
              },
              {
                label: "Urltest",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/universal/urltest.ini"
              }
            ]
          },
          {
            label: "customized",
            options: [
              {
                label: "Maying",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/maying.ini"
              },
              {
                label: "Ytoo",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/ytoo.ini"
              },
              {
                label: "FlowerCloud",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/flowercloud.ini"
              },
              {
                label: "Nexitally",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/nexitally.ini"
              },
              {
                label: "SoCloud",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/socloud.ini"
              },
              {
                label: "ARK",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/ark.ini"
              },
              {
                label: "ssrCloud",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/customized/ssrcloud.ini"
              }
            ]
          },
          {
            label: "Special",
            options: [
              {
                label: "NeteaseUnblock(仅规则，No-Urltest)",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/special/netease.ini"
              },
              {
                label: "Basic(仅GEOIP CN + Final)",
                value:
                  "https://cdn.jsdelivr.net/gh/SleepyHeeead/subconverter-config@master/remote-config/special/basic.ini"
              }
            ]
          }
        ]
      },
      form: {
        sourceSubUrl: "",
        clientType: "",
        customBackend: "",
        remoteConfig: "",
        excludeRemarks: "",
        includeRemarks: "",
        filename: "",
        emoji: true,
        nodeList: false,
        sort: false,
        udp: false,
        tfo: false,
        scv: true,
        fdn: false,
        expand: true,
        appendType: false,
        insert: false,
        new_name: true,
        tpl: {
          surge: {
            doh: false
          },
          clash: {
            doh: false
          }
        }
      },
      customParams: [],
      customSubUrl: "",
      needUdp: false,
    };
  },
  created() {
    document.title = "Subscription Converter";
    this.isPC = this.$getOS().isPc;
    if (process.env.VUE_APP_USE_STORAGE === 'true') {
      this.form.sourceSubUrl = this.getLocalStorageItem('sourceSubUrl')
    }
  },
  mounted() {
    this.form.clientType = "clash";
    // this.getBackendVersion();
  },
  beforeDestroy() {
  },
  methods: {
    onCopy() {
      this.$message.success("Copied!");
    },
    addCustomParam(){
      this.customParams.push({
        name: "",
        value: "",
      })
    },
    makeUrl() {
      if (this.form.sourceSubUrl === "" || this.form.clientType === "") {
        this.$message.error("订阅链接与客户端为必填项");
        return false;
      }

      let backend =
        this.form.customBackend === ""
          ? defaultBackend
          : this.form.customBackend;

      let sourceSub = this.form.sourceSubUrl;
      sourceSub = sourceSub.replace(/(\n|\r|\n\r)/g, "|");

      this.customSubUrl =
        backend +
        "target=" +
        this.form.clientType +
        "&url=" +
        encodeURIComponent(sourceSub) +
        "&insert=" +
        this.form.insert;

      if (this.advanced === "2") {
        if (this.form.remoteConfig) {
          this.customSubUrl +=
            "&config=" + encodeURIComponent(this.form.remoteConfig);
        }
        if (this.form.excludeRemarks) {
          this.customSubUrl +=
            "&exclude=" + encodeURIComponent(this.form.excludeRemarks);
        }
        if (this.form.includeRemarks) {
          this.customSubUrl +=
            "&include=" + encodeURIComponent(this.form.includeRemarks);
        }
        if (this.form.filename) {
          this.customSubUrl +=
            "&filename=" + encodeURIComponent(this.form.filename);
        }
        if (this.form.appendType) {
          this.customSubUrl +=
            "&append_type=" + this.form.appendType.toString();
        }

        this.customSubUrl +=
          "&emoji=" +
          this.form.emoji.toString() +
          "&list=" +
          this.form.nodeList.toString() +
          "&tfo=" +
          this.form.tfo.toString() +
          "&scv=" +
          this.form.scv.toString() +
          "&fdn=" +
          this.form.fdn.toString() +
          "&expand=" +
          this.form.expand.toString() +
          "&sort=" +
          this.form.sort.toString();

        if (this.needUdp) {
          this.customSubUrl += "&udp=" + this.form.udp.toString()
        }

        if (this.form.tpl.surge.doh === true) {
          this.customSubUrl += "&surge.doh=true";
        }

        if (this.form.clientType === "clash") {
          if (this.form.tpl.clash.doh === true) {
            this.customSubUrl += "&clash.doh=true";
          }
          this.customSubUrl += "&new_name=" + this.form.new_name.toString();
        }

        this.customParams.filter(param => param.name && param.value).forEach(param => {
          this.customSubUrl += `&${encodeURIComponent(param.name)}=${encodeURIComponent(param.value)}`
        })
      }

      this.$copyText(this.customSubUrl);
      this.$message.success("定制订阅已复制到剪贴板");
    },
    backendSearch(queryString, cb) {
      let backends = this.options.backendOptions;
      let results = queryString
        ? backends.filter(this.createFilter(queryString))
        : backends;
      cb(results);
    },
    createFilter(queryString) {
      return candidate => {
        return (
          candidate.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0
        );
      };
    },
    getBackendVersion() {
      this.$axios
        .get(
          defaultBackend.substring(0, defaultBackend.length - 5) + "/version"
        )
        .then(res => {
          this.backendVersion = res.data.replace(/backend\n$/gm, "");
          this.backendVersion = this.backendVersion.replace("subconverter", "");
        });
    },
    saveSubUrl() {
      if (this.form.sourceSubUrl !== '') {
        this.setLocalStorageItem('sourceSubUrl', this.form.sourceSubUrl)
      }
    },
    getLocalStorageItem(itemKey) {
      const now = +new Date()
      let ls = localStorage.getItem(itemKey)

      let itemValue = ''
      if (ls !== null) {
        let data = JSON.parse(ls)
        if (data.expire > now) {
          itemValue = data.value
        } else {
          localStorage.removeItem(itemKey)
        }
      }
      return itemValue
    },
    setLocalStorageItem(itemKey, itemValue) {
      const ttl = process.env.VUE_APP_CACHE_TTL
      const now = +new Date()

      let data = {
        setTime: now,
        ttl: parseInt(ttl),
        expire: now + ttl * 1000,
        value: itemValue
      }
      localStorage.setItem(itemKey, JSON.stringify(data))
    }
  },
};
</script>

<style scoped>
/* Glassmorphism Logic is in Main CSS, here we handle structure */
.main-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 20px;
  box-sizing: border-box;
}

.content {
  width: 100%;
  max-width: 900px;
}

.header {
  text-align: center;
  margin-bottom: 30px;
  color: #fff;
}

.title {
  font-size: 2.5rem;
  font-weight: 700;
  margin-bottom: 10px;
  text-shadow: 0 4px 6px rgba(0,0,0,0.1);
  letter-spacing: 1px;
}

.version {
  font-size: 0.9rem;
  opacity: 0.8;
  font-weight: 300;
  background: rgba(255,255,255,0.1);
  display: inline-block;
  padding: 4px 12px;
  border-radius: 20px;
  backdrop-filter: blur(5px);
}

.glass-card {
  background: rgba(255, 255, 255, 0.75);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border-radius: 24px;
  border: 1px solid rgba(255, 255, 255, 0.5);
  box-shadow: 0 16px 40px rgba(0, 0, 0, 0.05); /* Softer shadow */
  padding: 40px;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

@media (max-width: 768px) {
  .glass-card {
     padding: 20px;
  }
}

.section-title {
  font-size: 1.2rem;
  font-weight: 600;
  color: #333;
  margin-bottom: 15px;
  margin-top: 20px; /* Spacing between sections */
  display: flex;
  align-items: center;
}

.section-title:first-child {
  margin-top: 0;
}

.section-subtitle {
  font-size: 1rem;
  font-weight: 600;
  color: #555;
  margin-top: 15px;
  margin-bottom: 10px;
  padding-left: 5px;
  border-left: 3px solid #667eea;
}

.custom-radio-group .el-radio-button__inner {
  border: none;
  background: rgba(0,0,0,0.05);
  border-radius: 8px;
  margin-right: 10px;
  padding: 10px 20px;
  box-shadow: none;
  color: #555;
  transition: all 0.3s;
}

.custom-radio-group .el-radio-button__original-radio:checked + .el-radio-button__inner {
  background: #667eea;
  color: #fff;
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
}

/* Checkbox Grid */
.checkbox-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
  gap: 12px;
  margin-top: 10px;
}

/* Custom Param Structure */
.custom-param-row {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}
.param-name { width: 40%; }
.param-value { width: 60%; }

.add-param-btn-container {
  text-align: center;
  margin-top: 10px;
}

.divider {
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(0,0,0,0.1), transparent);
  margin: 40px 0;
}

.generate-btn-container {
  display: flex;
  justify-content: center;
  margin-top: 30px;
}

.generate-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  padding: 15px 50px;
  font-size: 1.1rem;
  font-weight: 600;
  border-radius: 50px;
  box-shadow: 0 10px 25px rgba(118, 75, 162, 0.4);
  transition: all 0.3s ease;
  width: 100%;
  max-width: 400px;
}

.generate-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 15px 35px rgba(118, 75, 162, 0.5);
}

.generate-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

/* Overriding Element UI for Glass Effect on specific inputs */
::v-deep .el-input__inner, ::v-deep .el-textarea__inner {
  background: rgba(255, 255, 255, 0.6) !important;
  border: 1px solid rgba(0, 0, 0, 0.05) !important;
  border-radius: 12px !important;
  backdrop-filter: blur(5px);
  transition: all 0.3s;
  color: #333;
}

::v-deep .el-input__inner:focus, ::v-deep .el-textarea__inner:focus {
  background: rgba(255, 255, 255, 0.95) !important;
  border-color: #667eea !important;
  box-shadow: 0 0 0 4px rgba(102, 126, 234, 0.1);
}

::v-deep .el-card {
  border: none;
  background: transparent;
}

::v-deep .el-checkbox.is-bordered {
  background: rgba(255, 255, 255, 0.5);
  border: 1px solid transparent;
  border-radius: 10px;
  width: 100%;
  margin-right: 0;
  padding: 10px 15px;
  height: auto;
  transition: all 0.2s;
}

::v-deep .el-checkbox.is-bordered.is-checked {
  background: rgba(102, 126, 234, 0.1);
  border-color: #667eea;
}

::v-deep .el-checkbox__label {
  font-weight: 500;
  color: #444;
}

/* Fix Element UI form item margin */
.el-form-item {
  margin-bottom: 22px;
}
</style>
