<template>
        <div>
          <div style="position: relative">
            <span style="position: absolute; right: 10px; top: 0px; z-index: 99" v-hasPermi="['system:lang:setting:button']">
              <el-tooltip
                class="item"
                effect="dark"
                :content="$t('sysytem.lang.setting.tip')"
                placement="top-end"
              >
                <img
                  class="el-icon-s-tools"
                  @click="openDialog"
                  :src="langSetting"
                  style="width: 30px"
                />
              </el-tooltip>
            </span>
            <slot></slot>
          </div>
          <Page
            :title="$t('dialogTip')"
            :backText="$t('system.back')"
            :cancelText="$t('system.cancel')"
            :okText="$t('system.save')"
            :visible.sync="dialogVisible"
            :backAction="backAction"
            :okAction="okAction"
            :cancelAction="
              () => {
                dialogVisible = false;
              }
            "
          >
            <div style="margin: 8px 10px">
              <el-table
                border
                :data="tableEditData"
                style="width: 100%"
                @cell-mouse-enter="handleCellEnter"
                @cell-mouse-leave="handleCellLeave"
              >
                <el-table-column
                  :prop="item"
                  :label="getChinaLang(item)"
                  v-for="item in formatTableEdit()"
                >
                  <div class="item" slot-scope="scope" :key="item">
                    <el-input
                      class="item__input"
                      :disabled="item === 'key' || item === 'name'"
                      v-model="scope.row[item]"
                      :placeholder="$t('common.enterTip')"
                    ></el-input>
                  </div>
                </el-table-column>
              </el-table>
            </div>
          </Page>
          <el-dialog
            :title="$t('system.good.tip')"
            :visible.sync="isShow"
            width="30%"
            center
          >
            <span>{{ $t("lang.change.tip") }}</span>
            <span slot="footer" class="dialog-footer">
              <el-button @click="isShow = false">{{ $t("system.cancel") }}</el-button>
              <el-button
                type="primary"
                @click="
                  () => {
                    dialogVisible = false;
                    isShow = false;
                  }
                "
                >{{ $t("system.sure") }}</el-button
              >
            </span>
          </el-dialog>
        </div>
      </template>

      <script>
      import { mapState } from "vuex";
      import LangSetting from "@/assets/lang/lang-setting.png";
      import Page from "./page.vue";

      import {
        getLangListByLang,
        getLangList,
        getPageTranslateData,
        savePageTranslateData,
      } from "@/api/i18";
      export default {
        props: ["fields", "model"],
        data() {
          return {
            isShow: false,
            dialogVisible: false,
            tableEditData: [],
            tableEditDataPre: "",
            langs: [],
            langSetting: LangSetting,
          };
        },
        computed: mapState({
          lang: (state) => state.lang.lang,
        }),
        watch: {
          lang(a, b) {
            this.init();
          },
        },
        components: {
          Page,
        },
        methods: {
          formatTableEdit() {
            if (!this.tableEditData[0]) {
              return [];
            }
            return Object.keys(this.tableEditData[0]).filter(
              (item) => item !== "model" && item !== "key"
            );
          },
          getChinaLang(lang) {
            return (
              this.langs.filter((value) => value.lang === lang)[0]?.name || "内容"
            );
          } /** 鼠标移入cell */,
          handleCellEnter(row, column, cell, event) {
            const property = column.property;
            if (this.editProp.includes(property)) {
              cell.querySelector(".item__input").style.display = "block";
              cell.querySelector(".item__txt").style.display = "none";
            }
          },
          /** 鼠标移出cell */
          handleCellLeave(row, column, cell, event) {
            const property = column.property;
            if (this.editProp.includes(property)) {
              cell.querySelector(".item__input").style.display = "none";
              cell.querySelector(".item__txt").style.display = "block";
            }
          },
          openDialog() {
            this.dialogVisible = true;
            this.init();
          },
          backAction() {
            if (JSON.stringify(this.tableEditData) !== this.tableEditDataPre) {
              this.isShow = true;
            } else {
              this.dialogVisible = false;
            }
          },
          okAction() {
            let params = this.tableEditData.map((item) => {
              return { ...item };
            });
            savePageTranslateData(params).then((res) => {
              this.init(() => {
                this.dialogVisible = false;
                this.$parent?.refresh && this.$parent?.refresh();
                this.$parent?.$parent?.refresh && this.$parent?.$parent?.refresh();
                this.$parent?.$parent?.$parent &&
                  this.$parent?.$parent?.$parent?.refresh();
              });
            });
          },
          init(callback) {
            let params = {
              model: this.$props.model,
              pageId: this.$route.name,
              fields: this.$props.fields,
            };
            this.getPageTranslateDataAction(params, callback);
          },
          concactI18(obj, callback) {
            this.$i18n.mergeLocaleMessage(this.lang, obj);
            this.$i18n.locale = this.lang;
            callback && callback();
          },

          getPageTranslateDataAction(params, callback) {
            getPageTranslateData(params).then((res) => {
              this.tableEditData = res.data;
              this.tableEditDataPre = JSON.stringify(res.data);
              let objInit = {};
              res.data.map((item) => {
                objInit[item.key] = item.name;
              });
              getLangListByLang({ lang: this.lang }).then((res) => {
                let obj = {};
                res.data.map((item) => {
                  obj[item.key] = item[this.lang];
                });
                this.concactI18({ ...objInit, ...obj }, callback);
              });
              getLangList().then((res) => {
                this.langs = res.data;
              });
            });
          },
          matchKey(name,value) {
            if(value){
              const langResult = this.tableEditData.filter((item) => item.name == name)?.[0]?.[this.lang]
              return langResult ? langResult : name;
            }
            return this.tableEditData.filter((item) => item.name == name)[0]?.key;
          },
          transRules(rules) {
            let newRules = {}
            for (let key in rules) {
              rules[key] = rules[key].map(item => {
                return {...item, message: this.matchKey(item.message, true)}
              })
              newRules[key]= rules[key]
            }
            return newRules;
          }
        },
        mounted() {
          this.init();
        },
      };
      </script>
      <style scoped lang="less">
      ::v-deep .el-dialog {
        padding: 0 !important;
      }
      </style>
