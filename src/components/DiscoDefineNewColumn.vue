<!--

This component is used app-wide to set up the properties when making a new column.

Its usage might look something like this:

  <discoDefineNewColumn
    v-show="addNewColumn"
    :columnName="generatedColumnName"
    :columnLabel="gene<!--

This component is used app-wide to set up the properties when making a new column.

Its usage might look something like this:

  <discoDefineNewColumn
    v-show="addNewColumn"
    :columnName="generatedColumnName"
    :columnLabel="generatedColumnLabel"
    @newColumnName="opData.newColumnName = $event"
    @newColumnLabel="opData.newColumnLabel = $event"
    @isValid="opData.isValid = $event"
  />

v-show: if the column name and label need to be persisted across visibility of this element and this state is not handled in the parent component, prefer v-show to v-if
columnName, columnLabel: these props are for the initial column name and label. These are different for each op so they should be generated parent-side as computed properties. See opOutlierDetection for a working example.
@newColumnName, @newColumnLabel: Emit events with the final column name and label that should be used in the op's data
@isValid: Emits boolean of current properties' collective validity

-->

<template>
<div>
  <label for="newColumnName" class="inputLabel caption">NEW COLUMN NAME</label>
  <div class="fakeElInputContainer">
    <i
      class="el-icon-circle-close clearIcon"
      v-if="clearIcon"
      @mouseover="inputHovered = true"
      @click="clearInput"
    />
    <input
      ref="newColumnName"
      id="newColumnName"
      placeholder="Name of column"
      :value="newColumnName"
      clearable
      required
      class="fakeElInput"
      :class="fakeElInputClass"
      @focus="handleFocus"
      @mouseover="inputHovered = true"
      @mouseleave="inputHovered = false"
      @blur="handleNewColumnInput($event, 'newColumnName')"
    />
  </div>
  <template v-if="$v.newColumnName">
    <p v-if="$v.newColumnName.required === false" class="errorMessage caption">required field</p>
    <p v-else-if="!$v.newColumnName.patternMatch" class="errorMessage caption">
      Column names must start with a letter and cannot end with an underscore.
      <br />
      They may contain only lowercase alphanumeric characters and underscores.
    </p>
  </template>

  <label for="newColumnLabel" class="inputLabel caption">NEW COLUMN LABEL</label>
  <el-input
    id="newColumnLabel"
    placeholder="Label of column"
    :value="newColumnLabel"
    clearable
    required
    :class="{ invalid: $v.newColumnLabel.$error, pulse: shouldPulse }"
    @blur="handleNewColumnInput($event, 'newColumnLabel')"
  />
  <p v-if="!$v.newColumnLabel.required" class="errorMessage caption">
    required field
  </p>
</div>
</template>

<script>
import { required } from 'vuelidate/lib/validators'

export default {
  validations() {
    if (this.shouldClear) {
      this.newColumnName = ''
      return { newColumnLabel: { required } }
    }

    return {
      newColumnName: {
        required,
        patternMatch: input => /^[a-z]([a-z0-9_]*[a-z0-9])?$/.test(input)  // all characters must be lowercase alphanumeric/underscore, first char must be letter, last cannot be underscore
      },
      newColumnLabel: { required }
    }
  },

  props: {
    columnName: {
      type: String,
      required: true
    },
    columnLabel: {
      type: String,
      required: true
    }
  },

  data() {
    return {
      newColumnName: this.columnName,
      newColumnLabel: this.columnLabel,

      unwatchValidation: '',
      shouldAutocorrect: true,
      shouldPulse: false,

    // Fake el-input:
      inputFocused: false,
      inputHovered: false,
      // This tracks whether Vuelidation should run or not on the name input field
      shouldClear: false  // If the field is cleared but still focused it shouldn't be Vuelidated yet
    }
  },

  computed: {
    clearIcon() {
      return (this.inputFocused || this.inputHovered)
    },

    fakeElInputClass() {
      const css = {
        invalid: false,
        pulse: this.shouldPulse
      }

      if (this.$v.newColumnName) {
        if(this.$v.newColumnName.$error || (!this.shouldClear && !this.newColumnName)) {
          css.invalid = true
        }
      }

      return css
    }
  },

  created() {
    // Emit boolean reflecting current validation state:
    this.unwatchValidation = this.$watch('$v.$invalid', e => {
      this.$emit('isValid', !e)
    })
  },

  beforeDestroy() {
    this.unwatchValidation()
  },

  watch: {
    columnName(newName) {
      if(!this.$v.newColumnName.$dirty) {
        this.newColumnName = newName
        this.emitNewName()
      }
    },
    columnLabel(newLabel) {
      if(!this.$v.newColumnLabel.$dirty) {
        this.newColumnLabel = newLabel
        this.emitNewLabel()
      }
    }
  },

  methods: {
    autoCorrectNewColumn() {
      if(this.shouldAutocorrect && this.$v.$anyDirty) {
        // Ensure this function runs only once when the user touched neither new column field
        this.shouldAutocorrect = false

        // If it's the name that's been changed,
        if(this.$v.newColumnName.$dirty && !this.$v.newColumnName.patternMatch) {
          // And the user hasn't changed it to an empty string
          if (this.$v.newColumnName.$model) {
            // Trigger pulse animations to signal to the user some change is happening:
            this.shouldPulse = true

            // Set the column LABEL to the column NAME's invalid input:
            this.$v.newColumnLabel.$model = this.$v.newColumnName.$model

            // Autocorrect newColumnName:
            this.$v.newColumnName.$model = this.$v.newColumnName.$model.toLowerCase()  // must be lowercase
                                                           .replace(/ +/g, '_')          // spaces become underscores
                                                           .replace(/[^a-z\d_]/g, '')    // letters, numbers, underscores
                                                           .replace(/^[^a-z]*/, '')      // must start with a letter
                                                           .replace(/_*$/, '')           // may not end with underscore

            this.emitAllInput()
          }
        }
      } else {
        // Remove pulse animations for all future checks:
        this.shouldPulse = false
      }
    },

    clearInput() {
      this.shouldClear = true
      this.$refs.newColumnName.focus()
    },

    emitAllInput() {
      this.emitNewName()
      this.emitNewLabel()
    },

    emitNewLabel() {
      this.$emit('newColumnLabel', this.newColumnLabel)
    },

    emitNewName() {
      this.$emit('newColumnName', this.newColumnName)
    },

    handleFocus() {
      this.inputFocused = true
      this.$emit('focus')
    },

    handleNewColumnInput(e, inputField) {
      this.inputFocused = false

      const newInput = e.target.value

      this[inputField] = newInput

      // Mark field dirty if not cleared && still focused name input field
      if(this.$v[inputField]) this.$v[inputField].$touch()

      if(!this.shouldClear) this.autoCorrectNewColumn()
      this.shouldClear = false

      // emit new name or label
      this.$emit(inputField, this[inputField])

      // Definitely mark dirty @blur if not marked earlier
      this.$v[inputField].$touch()
    }
  }
}
</script>

<style lang="scss" scoped>
.fakeElInputContainer {
  position: relative;

  @at-root.clearIcon {
    position: absolute;
    top: 12.5px;
    left: calc(100% - 25px);
    // font-size: 15px;
    width: 25px;
    color: #c0c4cc;

    &:hover {
      color: rgb(144, 157, 153);
    }
  }

  @at-root.fakeElInput {
    width: 100%;
    background-color: #fff;
    border-radius: 4px;
    border: 1px solid #dcdfe6;
    color: #606266;
    display: inline-block;
    height: 40px;
    line-height: 1;
    padding: 0 15px;
    transition: 'border-color .2s cubic-bezier(.645,.045,.355,1)';
    font-size: 14px;

    &::placeholder {
      color: #c0c4cc;
    }

    &:hover {
      border-color: #c0c4cc;
    }

    &:focus {
      outline: none;
      border-color: rgb(145, 192, 242);
    }

    &.invalid {
      border: 1px solid pink;
    }
  }
}

.maxWidth {
  width: 100%;
}

.inputLabel {
  display: block;
  padding-top: 36px;
  padding-bottom: 6px;
  text-transform: uppercase;
}

.invalid {
  /deep/ .el-input__inner {
    border: 1px solid pink;
  }
}

.errorMessage {
  color: pink;
  position: absolute;
  margin-top: 3px;
}

// Pulse animations:
.pulse {
  animation: pulse 1s 1;  // Run only one time
}

@keyframes pulse {
  0% {
    box-shadow: 0 0 0 0 rgba(pink, 0.4)
  }
  70% {
      box-shadow: 0 0 0 10px rgba(pink, 0);
  }
  100% {
      box-shadow: 0 0 0 0 rgba(pink, 0);
  }
}
</style>