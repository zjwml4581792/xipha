<!--
 * @Autor: zhenjun
 * @Date: 2024-09-30 14:08:31
 * @LastEditors: zhenjun
 * @LastEditTime: 2024-09-30 15:59:11
 * @Description: 
-->
<template>
  <div>
    <t-dialog
      v-model:visible="formData.backpackVisible"
      header="请选择结晶回路"
      width="60%"
      :show-all-levels="false"
      :on-confirm="onConfirmOrbment"
    >
      <t-cascader v-model="formData.orbmentSelect" :options="orbmentOptions">
        <template #option="{ item }">
          <div class="orbment-option">
            <div class="diamond-container">
              <div :class="'diamond-' + item.type"></div>
              <div :class="'inner-diamond-' + item.type"></div>
            </div>
            <div class="orbment-option-text">{{ item.label }}</div>
          </div>
        </template>
      </t-cascader>
    </t-dialog>
    <t-dialog
      v-model:visible="formData.shadowSkillVisible"
      header="晶片技能"
      width="60%"
      :footer="false"
      :show-all-levels="false"
    >
      <t-row>
        <t-col v-for="(ab,ind) in [0,1]" :key="ind" :span="6">
          <t-list>
            <template v-for="(shadow, shadowIndex) in shadowList" :key="shadow.id">
              <t-list-item
                :class="
                  'bg-color' +
                  checkOrbment(
                    formData.useList[0][shadow.cost[0].type],
                    shadow.cost[0].price
                  ) +
                  '-' +
                  shadow.cost[0].type
                "
                v-if="shadowIndex % 2 === ab"
              >
                <template #content>
                  <div class="shadow-list-item">
                    <div class="shadow-name">{{ shadow.name }}</div>
                    <div class="shadow-cost">
                      <div class="cost">
                        <div :class="'cost-type-container-' + shadow.cost[0].type">
                          <div :class="'cost-type-' + shadow.cost[0].type">
                            {{ orbmentMap[shadow.cost[0].type] }}
                          </div>
                        </div>
                        <div
                          :class="
                            'cost-use-' +
                            checkOrbment(
                              formData.useList[0][shadow.cost[0].type],
                              shadow.cost[0].price
                            )
                          "
                        >
                          {{ formData.useList[0][shadow.cost[0].type] }}
                        </div>
                        <div>/</div>
                        <div class="cost-price">
                          {{ shadow.cost[0].price }}
                        </div>
                      </div>
                      <div class="cost" v-if="shadow.cost.length > 1">
                        <div :class="'cost-type-container-' + shadow.cost[1].type">
                          <div :class="'cost-type-' + shadow.cost[1].type">
                            {{ orbmentMap[shadow.cost[1].type] }}
                          </div>
                        </div>
                        <div
                          :class="
                            'cost-use-' +
                            checkOrbment(
                              formData.useList[1][shadow.cost[1].type],
                              shadow.cost[1].price
                            )
                          "
                        >
                          {{ formData.useList[1][shadow.cost[1].type] }}
                        </div>
                        <div>/</div>
                        <div class="cost-price">
                          {{ shadow.cost[1].price }}
                        </div>
                      </div>
                    </div>
                  </div>
                </template>
              </t-list-item>
            </template>
          </t-list>
        </t-col>
      </t-row>
    </t-dialog>
    <t-space direction="vertical" size="large">
      <t-card v-for="(linkName, index) in linkNames" :key="index" :title="linkName">
        <template #actions>
          <t-button theme="primary" variant="outline" @click="onClickShadow(linkName)"
            >晶片技能</t-button
          >
        </template>
        <t-form
          ref="form"
          :data="formData"
          label-width="120px"
          layout="inline"
          scroll-to-first-error="smooth"
        >
          <template
            v-for="(pos, j) in [index * 4, index * 4 + 1, index * 4 + 2, index * 4 + 3]"
            :key="j"
          >
            <t-form-item :name="linkName + pos">
              <template #label>
                <t-select
                  v-model="formData.holeList[pos].type"
                  :style="
                    formData.holeList[pos].type
                      ? 'color:' + orbmentColor[formData.holeList[pos].type]
                      : ''
                  "
                >
                  <t-option
                    v-for="item in orbmentType"
                    :key="item.key"
                    :style="'color:' + item.color"
                    :label="item.name"
                    :value="item.key"
                  >
                  </t-option>
                </t-select>
              </template>
              <div class="circuit" @click="onClickCircuit(pos)">
                <div class="diamond-container">
                  <div :class="'diamond-' + formData.orbmentList[pos].type"></div>
                  <div :class="'inner-diamond-' + formData.orbmentList[pos].type"></div>
                </div>
                <div class="circuit-icon-text">{{ formData.orbmentList[pos].name }}</div>
              </div>
            </t-form-item>
          </template>
        </t-form>
      </t-card>
    </t-space>
  </div>
</template>
<script setup>
import { reactive, onBeforeMount, onMounted, computed } from "vue";
import { orbmentMap, orbmentColor, orbmentType } from "@/assets/data/orbment";
import orbmentList from "@/assets/data/kuro2_orbment.json";
import skillList from "@/assets/data/kuro2_skill.json";

const linkNames = ["武器", "护盾", "驱动", "EXTRA"];

const formData = reactive({
  backpackVisible: false,
  shadowSkillVisible: true,
  shadowType: "武器",
  orbmentSelect: "",
  holeSelect: {
    type: "all",
    name: "无限定",
    index: -1,
  },
  holeList: [],
  orbmentList: [],
  useList: [],
});

const checkOrbment = (a, b) => {
  if (a >= b) {
    return "success";
  } else {
    return "failure";
  }
};

const shadowList = computed(() => {
  let result = [];
  for (let i = 0; i < skillList.length; i++) {
    if (formData.shadowType === skillList[i].position) {
      result.push(skillList[i]);
    }
  }
  return result;
});

// 一个计算属性 ref
const orbmentOptions = computed(() => {
  let result = [];
  for (let i = 2; i < orbmentType.length; i++) {
    if ("all" === formData.holeSelect.type) {
      result.push({
        label: orbmentType[i].name,
        value: orbmentType[i].key,
        type: orbmentType[i].key,
        children: [],
      });
    } else {
      result.push({
        label: orbmentType[i].name,
        value: orbmentType[i].key,
        type: orbmentType[i].key,
        children: [],
      });
    }
  }
  for (let i = 0; i < orbmentList.length; i++) {
    if ("earth" === orbmentList[i].type) {
      result[0].children.push({
        label: orbmentList[i].name,
        value: orbmentList[i].id,
        type: orbmentList[i].type,
      });
    } else if ("water" === orbmentList[i].type) {
      result[1].children.push({
        label: orbmentList[i].name,
        value: orbmentList[i].id,
        type: orbmentList[i].type,
      });
    } else if ("fire" === orbmentList[i].type) {
      result[2].children.push({
        label: orbmentList[i].name,
        value: orbmentList[i].id,
        type: orbmentList[i].type,
      });
    } else if ("wind" === orbmentList[i].type) {
      result[3].children.push({
        label: orbmentList[i].name,
        value: orbmentList[i].id,
        type: orbmentList[i].type,
      });
    } else if ("time" === orbmentList[i].type) {
      result[4].children.push({
        label: orbmentList[i].name,
        value: orbmentList[i].id,
        type: orbmentList[i].type,
      });
    } else if ("gold" === orbmentList[i].type) {
      result[5].children.push({
        label: orbmentList[i].name,
        value: orbmentList[i].id,
        type: orbmentList[i].type,
      });
    } else if ("silver" === orbmentList[i].type) {
      result[6].children.push({
        label: orbmentList[i].name,
        value: orbmentList[i].id,
        type: orbmentList[i].type,
      });
    }
  }
  return result;
});

const onClickCircuit = (circuitIndex) => {
  formData.backpackVisible = true;
  formData.holeSelect = {
    type: formData.holeList[circuitIndex].type,
    index: circuitIndex,
  };
  console.log("circuitIndex", circuitIndex);
};

const onConfirmOrbment = () => {
  formData.orbmentList[formData.holeSelect.index] =
    orbmentList[parseInt(formData.orbmentSelect) - 1];
  formData.orbmentSelect = {
    type: "all",
    name: "无限定",
    index: -1,
  };
  formData.backpackVisible = false;
};

const onClickShadow = (linkName) => {
  formData.shadowSkillVisible = true;
  formData.shadowType = linkName;
};

onBeforeMount(() => {
  for (let i = 0; i < 16; i++) {
    formData.holeList.push({
      type: "all",
      name: "通用孔",
    });
  }

  for (let i = 0; i < 16; i++) {
    formData.orbmentList.push({
      type: "none",
      name: "请选择",
    });
  }

  for (let i = 0; i < 4; i++) {
    formData.useList.push({
      earth: 5,
      water: 5,
      fire: 2,
      wind: 0,
      time: 6,
      gold: 8,
      silver: 5,
    });
  }
});

onMounted(() => {
  console.log("onMounted");
});
</script>
<style scoped lang="scss">
@import url("@/assets/orbment.scss");
@import url("./KurotwoView.scss");
</style>
