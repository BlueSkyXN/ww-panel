<template>
  <v-container>
    <h1 class="text-center mb-6">攻击x暴击x元素伤害的基值计算器</h1>

    <v-row>
      <v-col cols="12" md="6">
        <v-card v-for="(group, groupIndex) in parameterGroups.slice(0, 3)" :key="groupIndex" class="mb-6">
          <v-card-title>{{ group.title }}</v-card-title>
          <v-card-text>
            <v-row>
              <v-col cols="4" v-for="(field, fieldIndex) in group.fields" :key="fieldIndex">
                <v-text-field
                  :label="field.label"
                  v-model.number="field.value"
                  type="number"
                  :step="field.step"
                  outlined
                  dense
                ></v-text-field>
              </v-col>
            </v-row>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="6">
        <v-card v-for="(group, groupIndex) in parameterGroups.slice(3)" :key="groupIndex" class="mb-6">
          <v-card-title>{{ group.title }}</v-card-title>
          <v-card-text>
            <v-row>
              <v-col cols="4" v-for="(field, fieldIndex) in group.fields" :key="fieldIndex">
                <v-text-field
                  :label="field.label"
                  v-model.number="field.value"
                  type="number"
                  :step="field.step"
                  outlined
                  dense
                ></v-text-field>
              </v-col>
            </v-row>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <v-card class="mt-6">
      <v-card-title>计算结果</v-card-title>
      <v-card-text>
        <v-row>
          <v-col cols="12" sm="4" md="3" lg="2" v-for="(result, index) in calculatedResults" :key="index">
            <v-sheet rounded elevation="1" class="pa-4">
              <div class="text-subtitle-2">{{ result.label }}</div>
              <div class="text-h6">{{ result.value.toFixed(3) }}</div>
            </v-sheet>
          </v-col>
        </v-row>
      </v-card-text>
    </v-card>

    <v-card class="mt-6">
      <v-card-title>预测实际伤害基值提升率</v-card-title>
      <v-card-text>
        <v-row>
          <v-col cols="12" sm="4" md="3" lg="2" v-for="(improvement, index) in predictedImprovements" :key="index">
            <v-sheet rounded elevation="1" class="pa-4">
              <div class="text-subtitle-2">{{ improvement.label }}</div>
              <div class="text-h6">{{ (improvement.value * 100).toFixed(4) }}%</div>
            </v-sheet>
          </v-col>
        </v-row>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script>
export default {
  name: 'GameCalculator',
  data() {
    return {
      parameterGroups: [
        {
          title: '基本信息',
          fields: [
            { label: '初始暴击率', value: 0.05, step: 0.01 },
            { label: '初始暴击伤害', value: 1.50, step: 0.01 },
            { label: '角色攻击白值', value: 462, step: 1 },
            { label: '武器攻击白值', value: 587, step: 1 }
          ]
        },
        {
          title: '天赋',
          fields: [
            { label: '天赋暴击率', value: 0.08, step: 0.01 },
            { label: '天赋暴击伤害', value: 0.00, step: 0.01 },
            { label: '天赋攻击百分比', value: 0.12, step: 0.01 },
            { label: '天赋元素伤害百分比', value: 0.00, step: 0.01 }
          ]
        },
        {
          title: '武器',
          fields: [
            { label: '武器暴击率', value: 0.00, step: 0.01 },
            { label: '武器暴击伤害', value: 0.484, step: 0.01 },
            { label: '武器攻击百分比', value: 0.24, step: 0.01 },
            { label: '武器元素伤害百分比', value: 0.00, step: 0.01 }
          ]
        },
        {
          title: '声骸',
          fields: [
            { label: '声骸暴击率', value: 0.411, step: 0.01 },
            { label: '声骸暴击伤害', value: 1.136, step: 0.01 },
            { label: '声骸攻击百分比', value: 0.69, step: 0.01 },
            { label: '声骸攻击固定值', value: 390, step: 1 },
            { label: '声骸元素伤害百分比', value: 0.60, step: 0.01 }
          ]
        },
        {
          title: '其他',
          fields: [
            { label: '其他暴击率', value: 0.25, step: 0.01 },
            { label: '其他暴击伤害', value: 0.00, step: 0.01 },
            { label: '其他攻击百分比', value: 0.35, step: 0.01 },
            { label: '其他攻击固定值', value: 0.00, step: 1 },
            { label: '其他元素伤害百分比', value: 0.10, step: 0.01 }
          ]
        },
        {
          title: '技能参数',
          fields: [
            { label: '目标技能倍率', value: 12.1275, step: 0.01 },
            { label: '目标技能伤害百分比', value: 0.80, step: 0.01 },
            { label: '目标技能额外攻击百分比', value: 0.00, step: 0.01 },
            { label: '目标技能穿甲提升百分比', value: 0.30, step: 0.01 },
            { label: '目标元素伤害抗性百分比', value: 0.10, step: 0.01 }
          ]
        }
      ]
    };
  },
  computed: {
    finalCritRate() {
      return this.sumFieldValues('暴击率');
    },
    finalCritDamage() {
      return this.sumFieldValues('暴击伤害');
    },
    finalAttackPercent() {
      return this.sumFieldValues('攻击百分比');
    },
    finalAttackFixed() {
      return this.sumFieldValues('攻击固定值');
    },
    finalElementDamagePercent() {
      return this.sumFieldValues('元素伤害百分比');
    },
    skillMultiplier() {
      return this.getFieldValue('目标技能倍率');
    },
    skillDamagePercent() {
      return this.getFieldValue('目标技能伤害百分比');
    },
    skillExtraAttackPercent() {
      return this.getFieldValue('目标技能额外攻击百分比');
    },
    skillPenetrationPercent() {
      return this.getFieldValue('目标技能穿甲提升百分比');
    },
    elementResistancePercent() {
      return this.getFieldValue('目标元素伤害抗性百分比');
    },
    finalActualAttack() {
      const baseAttack = this.getFieldValue('角色攻击白值') + this.getFieldValue('武器攻击白值');
      return baseAttack * (1 + this.finalAttackPercent + this.skillExtraAttackPercent) + this.finalAttackFixed;
    },
    finalDamageBase() {
      return this.calculateDamageBase(this.finalCritRate, this.finalCritDamage, this.finalActualAttack, this.finalElementDamagePercent);
    },
    finalSkillDamage() {
      const totalDamagePercent = this.finalElementDamagePercent + this.skillDamagePercent;
      let damage = this.calculateDamageBase(this.finalCritRate, this.finalCritDamage, this.finalActualAttack, totalDamagePercent) * this.skillMultiplier;
      damage *= (1 + this.skillPenetrationPercent);
      damage *= (1 - this.elementResistancePercent);
      return damage;
    },
    calculatedResults() {
      return [
        { label: '最终暴击率', value: this.finalCritRate },
        { label: '最终暴击伤害', value: this.finalCritDamage },
        { label: '最终攻击百分比', value: this.finalAttackPercent + this.skillExtraAttackPercent },
        { label: '最终攻击固定值', value: this.finalAttackFixed },
        { label: '最终元素伤害百分比', value: this.finalElementDamagePercent },
        { label: '最终实际攻击', value: this.finalActualAttack },
        { label: '最终实际伤害基值', value: this.finalDamageBase },
        { label: '目标技能伤害', value: this.finalSkillDamage }
      ];
    },
    predictedImprovements() {
      return [
        { label: '提升1%暴击率', value: this.calculateImprovement('critRate', 0.01) },
        { label: '提升1%暴击伤害', value: this.calculateImprovement('critDamage', 0.01) },
        { label: '提升1%攻击百分比', value: this.calculateImprovement('attackPercent', 0.01) },
        { label: '提升10点攻击固定值', value: this.calculateImprovement('attackFixed', 10) },
        { label: '提升1%元素伤害百分比', value: this.calculateImprovement('elementDamagePercent', 0.01) }
      ];
    }
  },
  methods: {
    sumFieldValues(fieldName) {
      return this.parameterGroups.reduce((sum, group) => {
        const fields = group.fields.filter(f => f.label.includes(fieldName));
        return sum + fields.reduce((fieldSum, field) => fieldSum + field.value, 0);
      }, 0);
    },
    getFieldValue(fieldName) {
      for (const group of this.parameterGroups) {
        const field = group.fields.find(f => f.label === fieldName);
        if (field) return field.value;
      }
      return 0;
    },
    calculateDamageBase(critRate, critDamage, actualAttack, damagePercent) {
      return (actualAttack * critRate * critDamage + actualAttack * (1 - critRate)) * (1 + damagePercent);
    },
    calculateActualAttack(attackPercent, attackFixed) {
      const baseAttack = this.getFieldValue('角色攻击白值') + this.getFieldValue('武器攻击白值');
      return baseAttack * (1 + attackPercent + this.skillExtraAttackPercent) + attackFixed;
    },
    calculateImprovement(attribute, amount) {
      const originalDamage = this.finalSkillDamage;
      let newDamage;

      switch (attribute) {
        case 'critRate':
          newDamage = this.calculateNewDamage(this.finalCritRate + amount, this.finalCritDamage, this.finalActualAttack, this.finalElementDamagePercent + this.skillDamagePercent);
          break;
        case 'critDamage':
          newDamage = this.calculateNewDamage(this.finalCritRate, this.finalCritDamage + amount, this.finalActualAttack, this.finalElementDamagePercent + this.skillDamagePercent);
          break;
        case 'attackPercent': {
          const newAttack = this.calculateActualAttack(this.finalAttackPercent + amount, this.finalAttackFixed);
          newDamage = this.calculateNewDamage(this.finalCritRate, this.finalCritDamage, newAttack, this.finalElementDamagePercent + this.skillDamagePercent);
          break;
        }
        case 'attackFixed': {
          const newAttack = this.calculateActualAttack(this.finalAttackPercent, this.finalAttackFixed + amount);
          newDamage = this.calculateNewDamage(this.finalCritRate, this.finalCritDamage, newAttack, this.finalElementDamagePercent + this.skillDamagePercent);
          break;
        }
        case 'elementDamagePercent':
          newDamage = this.calculateNewDamage(this.finalCritRate, this.finalCritDamage, this.finalActualAttack, this.finalElementDamagePercent + this.skillDamagePercent + amount);
          break;
        default:
          newDamage = originalDamage;
      }

      return (newDamage - originalDamage) / originalDamage;
    },
    calculateNewDamage(critRate, critDamage, actualAttack, damagePercent) {
      let damage = this.calculateDamageBase(critRate, critDamage, actualAttack, damagePercent) * this.skillMultiplier;
      damage *= (1 + this.skillPenetrationPercent);
      damage *= (1 - this.elementResistancePercent);
      return damage;
    }
  }
};
</script>

<style scoped>
.v-sheet {
  transition: all 0.3s;
}
.v-sheet:hover {
  transform: translateY(-5px);
  box-shadow: 0 5px 15px rgba(0,0,0,0.1);
}
</style>