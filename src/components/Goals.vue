<template>
  <div class="goals">
    <div v-if="goals.goals.length === 0" class="goals-empty-message">
      <p>Goals track events in your log and show your progress.</p>
      <p>For a quick start, select one of the presets below.</p>
    </div>
    <div v-else class="goal-items-container">
      <goal-item v-for="g in goals.goals" :goal="g" @open="showGoalModal"></goal-item>
    </div>
    <el-dialog custom-class="modalContainer" v-model="goalModalVisible" @close="closeGoalModal" size="large" :close-on-click-modal="false">
      <goal-edit-modal :goal="goalCopy" @save="saveGoalModal" @close="closeGoalModal" @deleteGoal="deleteGoalModal"></goal-edit-modal>
    </el-dialog>
    <presets title="Add a goal" :items="goalPresets" @open="presetOpen"></presets>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import uuid from 'uuid'
import moment from 'moment'

import GoalEditModal from './Goals/GoalEditModal'
import GoalItem from './Goals/GoalItem'
import Presets from './Helpers/Presets'
import SpeedDial from './Utils/SpeedDial'

export default {
  components: {
    GoalEditModal,
    GoalItem,
    Presets,
    SpeedDial
  },
  computed: {
    ...mapGetters(['goals'])
  },
  methods: {
    createGoal(goal) {
      const id = uuid.v4()
      this.$store.dispatch('addGoal', Object.assign({
        id: id,
        title: 'New Goal',
        time_start: moment().startOf('day'),
        time_end: moment().add(1, 'month').endOf('month'),
        recurring_period: 'none',
        measurement_period: 'daily'
      }, goal))
      this.showGoalModal(id)
    },
    showGoalModal(id) {
      this.goalCopy = _.cloneDeep(_.find(this.goals.goals, g => g.id === id ))
      this.goalModalVisible = true
    },
    saveGoalModal() {
      this.$store.dispatch('updateGoal', this.goalCopy)
    },
    deleteGoalModal() {
      this.$store.dispatch('deleteGoal', this.goalCopy.id)
    },
    closeGoalModal() {
      this.goalModalVisible = false
      this.goalCopy = {}
    },
    presetOpen(goal) {
      this.createGoal(goal.goal)
    }
  },
  data() {
    return {
      goalModalVisible: false,
      goalCopy: {},
      goalPresets: [
        {
          title: 'Lose weight',
          icon: 'weight-scale',
          goal: {
            wizard: 'weight',
            title: 'Weight Loss',
            measurement_period: 'last_event',
            startingValue: 85,
            targetValue: 80,
            valueUnits: 'Kilogram',
            eventFilter: `function(event) {
              return event.title === "Weight Measurement";
            }`,
            eventReducer: {
              accumulator: {
                sum: 0,
                n: 0
              },
              iteratee: `function(accumulator, event) {
                accumulator.sum += event.fields.find(function(f){ return f.title.match(/weight/i) }).value;
                accumulator.n++;
                return accumulator;
              }`,
              reducer: `function(accumulator) {
                return Math.round(accumulator.sum / accumulator.n);
              }`
            }
          }
        }, {
          title: 'Exercise twice a week',
          icon: 'dumbbell',
          goal: {
            wizard: 'exercise',
            title: 'Exercise'
          }
        }, {
          title: 'Sleep well',
          icon: 'moon',
          goal: {
            wizard: 'sleep',
            title: 'Sleep Well'
          }
        }, {
          title: 'Eat diversively',
          icon: 'food',
          goal: {
            wizard: 'food',
            title: 'Eat diversively'
          }
        }, {
          title: 'Avoid stress',
          icon: 'calendar',
          goal: {
            wizard: 'stress',
            title: 'Avoid stress'
          }
        }, {
          title: 'Add custom goal',
          icon: 'plus',
          goal: {}
        }
      ]
    }
  }
}
</script>

<style>
.goals {
  height: 100%;
  overflow-y: scroll;
  background-color: #EDEFF4;
  padding: 24px;
}
.goals::-webkit-scrollbar {
  display: none;
}
.new-goal {
  position: absolute;
  bottom: 0;
  right: 0;
}
.goal-items-container {
  display: flex;
  flex-wrap: wrap;
}
@media screen and (max-width: 768px) {
  .goal-items-container {
    margin-top: 48px;
  }
}
.goal-items-container .goal-item {
  margin-right: 24px;
  margin-bottom: 24px;
}
.goals-empty-message {
  text-align: center;
  padding: 24px;
}
</style>
