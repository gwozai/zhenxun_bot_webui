<template>
  <div class="box-background">
    <div
      v-for="(plugin, index) in pluginEditList"
      :key="index"
      :class="{ pstatus: !plugin.plugin_manager.status, disabled: isOptReq }"
      class="card-box"
    >
      <div class="plugin-version" v-if="plugin.plugin_manager.version">
        {{ "v" + plugin.plugin_manager.version }}
      </div>
      <div class="author-box" :class="{ flexd: pluginType != 'normal' }">
        <div class="plugin-name">{{ plugin.plugin_manager.plugin_name }}</div>
        <div class="author-name" v-if="plugin.plugin_manager.author">
          {{ "@" + plugin.plugin_manager.author }}
        </div>
      </div>
      <div class="plugin-model-name">{{ "(" + plugin.model + ")" }}</div>
      <div class="options-box">
        <div
          class="plugin-status"
          :class="{ disabled: !isEdit[index] }"
          v-if="plugin.plugin_settings"
        >
          默认开关
          <div class="switch" style="margin-left: 1rem">
            <input
              :id="'switch-' + index"
              v-model="plugin.plugin_settings.default_status"
              type="checkbox"
            />
            <label :for="'switch-' + index"></label>
          </div>
        </div>
        <div
          class="plugin-switch"
          :class="{ disabled: !isEdit[index], pbnn: pluginType != 'normal' }"
        >
          <PlayButton
            :pluginSta="plugin.plugin_manager.status"
            :tindex="index"
            @func="getMsgFormPlayButton"
          ></PlayButton>
          <!-- <div class="m-left-1" v-show="plugin.plugin_manager.block_type">
                  <div class="block-type">禁用类型</div>
                  <SegmentedControl :block_type="plugin.plugin_manager.block_type" :tindex="index" @func="getMsgFormSegmentedControl"></SegmentedControl>
                </div> -->
        </div>
        <div
          class="plugin-super"
          :class="{ disabled: !isEdit[index] }"
          v-if="plugin.plugin_settings"
        >
          限制超级用户
          <div class="switch" style="margin-left: 1rem">
            <input
              :id="'switch1-' + index"
              v-model="plugin.plugin_settings.limit_superuser"
              type="checkbox"
            />
            <label :for="'switch1-' + index"></label>
          </div>
        </div>
        <div style="display: flex; align-items: center">
          <div
            class="plugin-cost"
            :class="{ disabled: !isEdit[index] }"
            v-if="plugin.plugin_settings"
          >
            花费
            <div class="form">
              <input
                type="text"
                v-model="plugin.plugin_settings.cost_gold"
                class="form__input"
                placeholder="0"
              />
            </div>
          </div>
          <div v-show="plugin.plugin_manager.block_type" class="block-type">
            禁用类型
          </div>
        </div>

        <div style="display: flex; align-items: center">
          <div
            class="plugin-type"
            :class="{ disabled: !isEdit[index] }"
            v-if="plugin.plugin_settings"
          >
            类型
            <div class="form">
              <input
                type="text"
                v-model="plugin.plugin_settings.plugin_type[0]"
                class="form__input"
                placeholder="无"
              />
            </div>
          </div>
          <SegmentedControl
            :pluginType="pluginType"
            v-show="plugin.plugin_manager.block_type"
            :block_type="plugin.plugin_manager.block_type"
            :tindex="index"
            @func="getMsgFormSegmentedControl"
          ></SegmentedControl>
        </div>

        <div
          class="plugin-other-name"
          :class="{ disabled: !isEdit[index] }"
          v-if="plugin.plugin_settings"
        >
          别名
          <div class="form">
            <input
              type="text"
              v-model="plugin.plugin_settings.cmd"
              class="form__input"
              placeholder="无"
            />
          </div>
        </div>
        <div
          class="plugin-Authority"
          :class="{ disabled: !isEdit[index] }"
          v-if="plugin.plugin_settings"
        >
          群权限
          <RateSlider
            :level="plugin.plugin_settings.level"
            :tindex="index"
            @func="getMsgFormRateSlider"
          ></RateSlider>
        </div>
      </div>

      <div
        v-if="!isEdit[index]"
        class="edit-btn"
        :class="{ 'no-setting': !plugin.plugin_settings }"
      >
        <div
          class="btn btn__primary"
          @click="
            showEditOpt(
              plugin.plugin_config,
              index,
              plugin.plugin_manager.plugin_name
            )
          "
          v-if="plugin.plugin_config"
        >
          <p>配置项</p>
        </div>
        <div
          class="btn btn__secondary"
          v-if="plugin.plugin_settings"
          @click="isEdit.splice(index, 1, true)"
        >
          <p>编辑</p>
        </div>
      </div>

      <div
        v-if="isEdit[index]"
        class="edit-btn"
        :class="{ 'no-setting': !plugin.plugin_settings }"
      >
        <div class="btn btn__primary" @click="doUpdate(index)">
          <p>保存</p>
        </div>
        <div class="btn btn__secondary" @click="cancelEdit(index)">
          <p>取消</p>
        </div>
      </div>
    </div>
    <EditOpt
      v-if="editOptShow"
      :class="{ disabled: isEditReq }"
      :editOptData="editOpt"
      @close="closeEditOptShow"
      @changeOpt="changeEditOpt"
    ></EditOpt>
  </div>
</template>

<script>
import RateSlider from "@/components/UI/RateSlider";
import PlayButton from "@/components/UI/PlayButton";
import SegmentedControl from "@/components/UI/SegmentedControl";
import EditOpt from "@/components/UI/EditOpt";

export default {
  name: "PluginListTemplate",
  components: {
    RateSlider,
    PlayButton,
    SegmentedControl,
    EditOpt,
  },
  props: ["pluginType"],
  data() {
    return {
      editOpt: {
        pluginName: "",
        data: {},
        index: -1,
      },
      isOptReq: false,
      isEditReq: false,
      editOptShow: false,
      isEdit: [],
      pluginSta: true,
      pluginEditList: [],
      pluginList: [],
      dialogVisible: false,
      configDialogVisible: false,
      pluginData: {
        model: "",
        plugin_settings: {
          level: 0,
          default_status: false,
          limit_superuser: null,
          cmd: "",
          cost_gold: 0,
          plugin_type: "",
        },
        plugin_manager: {
          plugin_name: "",
          status: null,
          error: null,
          version: 0,
          author: "",
          block_type: "all",
        },
        plugin_config: [],
        cd_limit: null,
        block_limit: null,
        count_limit: null,
      },
      configsType: [],
      blockType: [
        { value: "all", label: "全部" },
        { value: "group", label: "群组" },
        { value: "private", label: "私聊" },
      ],
    };
  },
  mounted() {
    this.initPluginList(true);
  },
  methods: {
    changeEditOpt(data) {
      this.isEditReq = true;

      this.postRequest("/zhenxun/api/update_config", data).then((resp) => {
        if (resp && resp.code == 200) {
          this.$message.success(resp.info);
          this.isEditReq = false;
          this.closeEditOptShow();
          this.initPluginList();
        } else {
          if (resp && resp.data) this.$message.error(resp.data);
          else this.$message.error("修改失败");
          this.isEditReq = false;
          // this.initPluginList();
        }
      });
    },
    showEditOpt(data, index, name) {
      this.editOpt.index = index;
      this.editOpt.data = data;
      this.editOptShow = true;
      this.editOpt.pluginName = name;
    },
    closeEditOptShow() {
      (this.editOpt = {
        pluginName: "",
        data: {},
        index: -1,
      }),
        (this.editOptShow = false);
    },
    cancelEdit(index) {
      // this.initPluginList();
      this.isEdit.splice(index, 1, false);
      this.pluginEditList.splice(
        index,
        1,
        JSON.parse(JSON.stringify(this.pluginList[index]))
      );
      // this.pluginEditList[index] = this.pluginList[index];
    },
    getMsgFormPlayButton(index, data) {
      this.pluginEditList[index].plugin_manager.status = data;
      if (
        this.pluginEditList[index].plugin_manager.status == false &&
        (this.pluginEditList[index].plugin_manager.block_type == "" ||
          this.pluginEditList[index].plugin_manager.block_type == null)
      ) {
        this.pluginEditList[index].plugin_manager.block_type = "全部";
      } else if (this.pluginEditList[index].plugin_manager.status == true) {
        this.pluginEditList[index].plugin_manager.block_type = "";
      }
    },
    getMsgFormSegmentedControl(index, data) {
      this.pluginEditList[index].plugin_manager.block_type = data;
    },
    getMsgFormRateSlider(index, data) {
      this.pluginEditList[index].plugin_settings.level = data;
    },
    refresh() {
      this.initPluginList();
    },
    initPluginList(isFirst = false) {
      this.getRequest(
        "/zhenxun/api/get_plugins?plugin_type=" + this.pluginType
      ).then((resp) => {
        if (resp) {
          this.$message.success(resp.info);
          this.pluginList = resp.data;
          if (isFirst) {
            this.pluginEditList = JSON.parse(JSON.stringify(this.pluginList));
            this.isEdit = new Array(this.pluginList.length).fill(false);
          }
        } else {
          this.$message.error("获取数据失败！");
        }
      });
    },
    showEditView(data) {
      this.pluginData = JSON.parse(JSON.stringify(data));
      if (
        this.pluginData.plugin_manager.block_type == "" ||
        this.pluginData.plugin_manager.block_type == null
      ) {
        this.pluginData.plugin_manager.block_type = "all";
      }
      this.dialogVisible = true;
    },
    doUpdate(index) {
      this.isOptReq = true;
      // 修复回传数据列表变成字符串的问题
      let postConfigData = JSON.parse(
        JSON.stringify(this.pluginEditList[index])
      ); //不提交原配置项而是一个副本
      console.log("postConfigData", postConfigData);
      let cmd = postConfigData?.plugin_settings?.cmd;
      if (cmd && !Array.isArray(cmd)) {
        cmd = cmd.split(",");
      }
      const sendData = {
        module: postConfigData.model,
        default_status: postConfigData?.plugin_settings?.default_status,
        limit_superuser: postConfigData?.plugin_settings?.limit_superuser,
        cost_gold: postConfigData?.plugin_settings?.cost_gold,
        cmd: cmd,
        menu_type: postConfigData?.plugin_settings?.plugin_type[0],
        group_level: postConfigData?.plugin_settings?.level,
        block_type: postConfigData?.plugin_manager?.status
          ? ""
          : postConfigData?.plugin_manager?.block_type,
      };
      console.log("sendData", sendData);
      this.postRequest("/zhenxun/api/update_plugins", sendData).then((resp) => {
        //先判断是否有返回数据
        if (resp && resp.code == 200) {
          this.$message.success(resp.info);
          // this.$message({
          //   message: "修改成功",
          //   type: "success",
          // });
          this.isOptReq = false;
          this.isEdit.splice(index, 1, false);
          this.initPluginList();
        } else {
          //修改失败也刷新一次配置
          if (resp && resp.data) this.$message.error(resp.data);
          else this.$message.error("修改失败");
          this.isOptReq = false;
          // this.initPluginList();
        }
      });
    },
    // doConfigUpdate() {
    //   let pluginConfigData = JSON.parse(JSON.stringify(this.pluginData));
    //   pluginConfigData.plugin_manager = null;
    //   pluginConfigData.plugin_settings = null;

    //   this.postRequest("/webui/plugins", pluginConfigData).then((resp) => {
    //     if (resp && resp.code == 200) {
    //       this.$message({
    //         message: "修改成功",
    //         type: "success",
    //       });
    //       this.initPluginList();
    //     } else {
    //        //修改失败也刷新一次配置
    //       if(resp && resp.data)
    //         this.$message.error(resp.data);
    //       else
    //         this.$message.error("修改失败");
    //       this.initPluginList();
    //     }
    //   });
    //   this.configDialogVisible = false;
    // },
  },
};
</script>

<style scoped>
.disabled {
  pointer-events: none;
  cursor: default;
}
.box-background {
  display: flex;
  width: 100%;
  height: 100%;
  justify-content: center;
  flex-flow: wrap;
  background: var(--greyLight-1);
}
.card-box {
  position: relative;
  width: 25rem;
  min-width: 25rem;
  /* height: 23rem; */
  min-height: 14rem;
  padding: 2rem;
  padding-bottom: 1rem;
  overflow: hidden;
  border-radius: 3rem;
  margin: 1rem;
  box-shadow: 0.8rem 0.8rem 1.4rem var(--greyLight-2),
    -0.2rem -0.2rem 1.8rem var(--white);
}
.plugin-version {
  position: absolute;
  line-height: 5rem;
  width: 10rem;
  height: 4rem;
  right: -3rem;
  top: 0rem;
  font-size: var(--titleH2);
  text-align: center;
  /* background-color: #2da44e; */
  background-color: var(--primary);
  transform: rotate(45deg);
  user-select: none;
}
.card-box.pstatus {
  --primary: hsla(356, 72%, 44%, 1);
  --primary-dark: rgba(207, 34, 46, 1);
}
.author-box {
  display: flex;
  height: var(--titleH1);
  line-height: var(--titleH1);
}
.flexd {
  flex-direction: column;
  height: auto;
  align-items: flex-start;
}
.author-name {
  margin-left: 0.5rem;
  color: #57606a;
  font-size: var(--content);
}
.plugin-name {
  font-size: var(--titleH1);
  font-weight: 700;
  color: #0969da;
}
.plugin-switch {
  position: absolute;
  right: 0.5rem;
  top: 3rem;
}
.pbnn {
  right: 0rem;
  top: 4rem;
}
.plugin-model-name {
  font-size: var(--titleH2);
  font-weight: 700;
  color: #0969da;
  text-align: start;
}
.options-box {
  min-height: 6rem;
}
.options-box > div {
  font-size: var(--contentP);
  margin: 0.8rem 0;
  user-select: none;
}
.edit-btn {
  display: flex;
  width: 100%;
  margin-top: 1rem;
  justify-content: space-around;
}
.edit-btn.no-setting {
  position: absolute;
  left: 0;
  bottom: 1rem;
}
.plugin-status,
.plugin-super {
  display: flex;
  align-items: center;
}
.block-type {
  text-align: start;
  height: var(--contentHeight);
  line-height: var(--contentHeight);
  margin-left: 1.5rem;
}
/*  BUTTONS  */
.btn {
  width: 10rem;
  height: 3rem;
  border-radius: 1rem;
  box-shadow: 0.3rem 0.3rem 0.6rem var(--greyLight-2),
    -0.2rem -0.2rem 0.5rem var(--white);
  justify-self: center;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  user-select: none;
  transition: 0.3s ease;
}

.btn__primary {
  background: var(--primary);
  box-shadow: inset 0.2rem 0.2rem 1rem var(--primary-light),
    inset -0.2rem -0.2rem 1rem var(--primary-dark),
    0.3rem 0.3rem 0.6rem var(--greyLight-2), -0.2rem -0.2rem 0.5rem var(--white);
  color: var(--greyLight-1);
}

.btn__primary:hover {
  color: var(--white);
}

.btn__primary:active {
  box-shadow: inset 0.2rem 0.2rem 1rem var(--primary-dark),
    inset -0.2rem -0.2rem 1rem var(--primary-light);
}

.btn__secondary {
  color: var(--greyDark);
}

.btn__secondary:hover {
  color: var(--primary);
}

.btn__secondary:active {
  box-shadow: inset 0.2rem 0.2rem 0.5rem var(--greyLight-2),
    inset -0.2rem -0.2rem 0.5rem var(--white);
}

.btn p {
  font-size: var(--contentBtn);
}

/*  SWITCH  */
.switch {
  width: 4rem;
}

.switch label {
  display: flex;
  align-items: center;
  width: 100%;
  height: var(--contentHeight);
  box-shadow: 0.3rem 0.3rem 0.6rem var(--greyLight-2),
    -0.2rem -0.2rem 0.5rem var(--white);
  background: rgba(255, 255, 255, 0);
  position: relative;
  cursor: pointer;
  border-radius: 1.6rem;
}

.switch label::after {
  content: "";
  position: absolute;
  left: 0.4rem;
  width: calc(var(--contentHeight) - 0.5rem);
  height: calc(var(--contentHeight) - 0.5rem);
  border-radius: 50%;
  background: var(--greyDark);
  transition: all 0.4s ease;
}

.switch label::before {
  content: "";
  width: 100%;
  height: 100%;
  border-radius: inherit;
  background: linear-gradient(
    330deg,
    var(--primary-dark) 0%,
    var(--primary) 50%,
    var(--primary-light) 100%
  );
  opacity: 0;
  transition: all 0.4s ease;
}

.switch input:checked ~ label::before {
  opacity: 1;
}

.switch input {
  display: none;
}

.switch input:checked ~ label::after {
  left: 50%;
  background: var(--greyLight-1);
}

/*  RANGE-SLIDER  */
.plugin-Authority {
  display: flex;
  width: 100%;
  align-items: center;
}
/*  FORM  */
.plugin-cost,
.plugin-other-name {
  display: flex;
  align-items: center;
}
.plugin-other-name .form__input {
  width: 100%;
}
.form {
  margin-left: 1rem;
  width: 69%;
}

.form__input {
  width: 6rem;
  height: var(--contentHeight);
  border: none;
  border-radius: 1rem;
  font-size: var(--contentP);
  padding: 0 1.4rem;
  box-shadow: 0.3rem 0.3rem 0.6rem var(--greyLight-2),
    -0.2rem -0.2rem 0.5rem var(--white);
  background: none;
  font-family: inherit;
  color: var(--greyDark);
}

.form__input::-moz-placeholder {
  color: var(--greyLight-3);
}

.form__input:-ms-input-placeholder {
  color: var(--greyLight-3);
}

.form__input::placeholder {
  color: var(--greyLight-3);
}

.form__input:focus {
  outline: none;
  box-shadow: inset 0.2rem 0.2rem 0.5rem var(--greyLight-2),
    inset -0.2rem -0.2rem 0.5rem var(--white);
}

/*  CHIP  */
.plugin-type {
  display: flex;
  align-items: center;
  margin-right: 1rem;
}
.chip {
  margin-left: 1rem;
  justify-self: center;
  width: 9rem;
  height: var(--contentHeight);
  border-radius: 1rem;
  box-shadow: 0.3rem 0.3rem 0.6rem var(--greyLight-2),
    -0.2rem -0.2rem 0.5rem var(--white);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.chip p {
  font-size: var(--content);
  padding-left: 1rem;
  color: var(--greyDark);
}
.chip__close {
  width: 2rem;
  height: 1rem;
  display: flex;
  font-size: var(--content);
  margin-right: 0.5rem;
  color: var(--greyLight-3);
  cursor: pointer;
  justify-content: center;
  align-items: center;
}
/* .m-left-1{
  margin-left: -1rem;
} */

@media screen and (max-width: 600px) {
  .box-background {
    --titleH1: 1rem;
    --titleH2: 0.7rem;
    --contentP: 0.65rem;
    --contentBtn: 0.8rem;
    --content: 0.5rem;
    --contentHeight: 1.5rem;
  }
  .card-box {
    position: relative;
    width: 80%;
    min-width: 80%;
    /* height: 18.5rem; */
    min-height: 11rem;
    padding: 1.5rem 1.5rem 1rem 1.5rem;
    margin: 1rem 0 0 0;
  }
  .card-box:nth-last-child(1) {
    margin: 1rem 0 1rem 0;
  }
  .plugin-other-name .form__input {
    width: 80%;
  }
  .switch {
    width: 2.8rem;
  }
  .plugin-version {
    height: 3rem;
    line-height: 3.5rem;
    width: 8rem;
    right: -2.5rem;
  }
  .btn {
    width: 35%;
    height: 2.5rem;
  }
  .plugin-switch {
    right: 1rem;
    top: 3rem;
  }
  .form__input {
    width: 3rem;
  }
  .options-box {
    min-height: 4rem;
  }
  .form {
    flex: 1;
    margin-left: 0.5rem;
  }
  /* .m-left-1{
    margin: 0;
  } */
}
</style>