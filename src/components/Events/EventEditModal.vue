<template>
  <div class="modal" @keyup.enter="modalOnEnter">
    <div class="headerButtons">
      <el-dropdown class="modalEditActionsDropdown" trigger="click" @command="eventAction">
        <el-button class="el-button--link" size="small">
          <i class="el-icon-more"></i>
        </el-button>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item command="duplicate"><i class="el-icon-plus"></i>Duplicate</el-dropdown-item>
          <el-dropdown-item command="delete"><i class="el-icon-delete"></i>Delete</el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>
    <div class="eventModalHeader">
      <el-input class="eventModalTitle" placeholder="Event title" v-model="event.title"></el-input>
      <p>
        <el-date-picker
          v-model="event.time"
          type="datetime"
          :clearable="false">
        </el-date-picker>
      </p>
    </div>
    <div v-for="(f, i) in event.fields">
      <field-view
        :field="f" view="modal"
        :key="f.id"
        @deleteField="deleteField(f.id)"
        >
      </field-view>
    </div>
    <el-dropdown class="modalEditAddField" trigger="click" @command="addField">
      <el-button type="primary" size="small" :plain="true">
        <i class="el-icon-plus"></i> Add Field
      </el-button>
      <el-dropdown-menu slot="dropdown">
        <el-dropdown-item command="length">Distance</el-dropdown-item>
        <el-dropdown-item command="duration">Duration</el-dropdown-item>
        <el-dropdown-item command="intensity">Intensity</el-dropdown-item>
        <el-dropdown-item command="select">Select</el-dropdown-item>
        <el-dropdown-item command="rank_stars">Stars</el-dropdown-item>
        <el-dropdown-item command="switch">Switch</el-dropdown-item>
        <el-dropdown-item command="text">Text</el-dropdown-item>
        <el-dropdown-item command="units">Units</el-dropdown-item>
        <el-dropdown-item command="weight">Weight</el-dropdown-item>
      </el-dropdown-menu>
    </el-dropdown>

    <div class="close-modal-button">
      <el-button @click="close" type="text" size="small" :plain="true">Cancel</el-button>
      <el-button @click="save" type="success">Save</el-button>
    </div>
  </div>
</template>

<script>
import FieldView from './FieldView'
import uuid from 'uuid'

export default {
  props: ['event'],
  components: {
    FieldView
  },
  data() {
    return {
      editMode: false
    }
  },
  computed: {
    hasFields() {
      return this.event.fields && this.event.fields.length > 0
    }
  },
  methods: {
    save() {
      this.$emit('save')
    },
    close() {
      this.$emit('close')
    },
    modalOnEnter(ev) {
      if (ev.target.tagName.toLowerCase() !== 'textarea') {
        this.close()
      }
    },
    addField(field) {
      this.event.fields.push(Object.assign({
        id: uuid.v4(),
        title: this.fieldType(field),
        type: field
      }, this.defaultValue(field)))
    },
    fieldType(type) {
      switch (type) {
        case 'text':
          return 'Text'
          break;
        case 'duration':
          return 'Duration'
          break;
        case 'intensity':
          return 'Intensity'
          break;
        case 'weight':
          return 'Weight'
          break;
        case 'length':
          return 'Length'
          break;
        case 'rank_stars':
          return 'Rank'
          break;
        case 'switch':
          return 'Switch'
          break;
        case 'select':
          return 'Select'
          break;
        case 'units':
          return 'Units'
          break;
        default:
          return type
      }
    },
    defaultValue(type) {
      switch (type) {
        case 'text':
          return { title: 'Notes', value: '' }
          break;
        case 'duration':
          return { value: 0 }
          break;
        case 'intensity':
          return { value: 1, options: { options: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10], customOptions: false } }
          break;
        case 'weight':
          return { value: 70, options: { units: 'Kilogram' } }
          break;
        case 'length':
          return { value: 5, options: { units: 'Meter' } }
          break;
        case 'rank_stars':
          return { value: 2.5 }
          break;
        case 'switch':
          return { value: false }
          break;
        case 'select':
          return { value: '', options: { options: [] } }
          break;
        case 'units':
          return { value: 0, options: { units: '' } }
          break;
        default:
          return {}
      }
    },
    deleteField(id) {
      const i = _.findIndex(this.event.fields, e => e.id === id)
      if (i > -1) {
        this.event.fields.splice(i, 1)
      }
    },
    eventAction(action) {
      switch(action) {
        case 'delete':
          this.deleteEvent()
          break;
        case 'duplicate':
          this.$emit('duplicateEvent', this.event)
          this.close()
          break;
        default:
          console.error('Unknown action', action)
      }
    },
    deleteEvent() {
      this.event._isDeleted = true
      this.close()
      this.$store.dispatch('deleteEvent', this.event)
    }
  }
}
</script>

<style>
.editFields {
  font-size: 14px;
}
.field-edit-main {
  margin-top: 30px;
  display: flex;
}
.field-edit-main > *:not(.field-title):not(.field-type) {
  font-size: 10px;
  flex-grow: 0;
  align-self: flex-end;
  margin: 0;
  margin-bottom: 6px;
  padding: 6px;
}
.field-edit-main .field-type {
  margin: 0 6px;
  flex-grow: 0;
}
.field-edit-main .field-title {
  margin: 0 6px;
  flex-grow: 2;
}
.field-edit-main .field-type-description {
  font-size: 12px;
  color: #838C91;
}
.modalEditAddField {
  margin-top: 36px;
}
.modalEditActionsDropdown {
  display: flex;
  justify-content: flex-end;
}
</style>
