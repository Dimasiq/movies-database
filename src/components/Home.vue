<template lang="pug">
  .content-wrapper
    section
      .container
        h1.ui-title-1 Home
        form(@submit.prevent="onSubmit")
          .form-item(:class="{ errorInput: $v.taskTitle.$error }")
            input(
              type="text"
              placeholder="What will we watch today?"
              v-model="taskTitle"
              @change="$v.taskTitle.$touch()"
              :class="{ error: $v.taskTitle.$error }"
            )
            .error(v-if="!$v.taskTitle.required") Title is required.
          .form-item
            textarea(
              type="text"
              v-model="taskDescription"
              @keyup.enter="onSubmit"
            )

          .option-list
            input.what-watch--radio(
              type="radio"
              id="radioMovie"
              value="movie"
              v-model="whatWatch"
            )
            label(
              for="radioMovie"
            ) Movie
            input.what-watch--radio(
              type="radio"
              id="radioSeries"
              value="series"
              v-model="whatWatch"
            )
            label(
              for="radioSeries"
            ) Series
          .total-time
            .total-time__movie(
              v-if="whatWatch === 'movie'"
            )
              span.time-title Hours
              input.time-input(
                type="number"
                v-model="movieHours"
              )
              span.time-title Minutes
              input.time-input(
                type="number"
                v-model="movieMinutes"
              )
              p {{ movieTime }}
            .total-time__series(
              v-if="whatWatch === 'series'"
            )
              span.time-title Seasons
              input.time-input(
                type="number"
                v-model="seriesSeasons"
              )
              span.time-title Episodes
              input.time-input(
                type="number"
                v-model="seriesEpisodes"
              )
              span.time-title Episode length (min)
              input.time-input(
                type="number"
                v-model="seriesEpisodeLength"
              )
              p {{ seriesTime }}
          .tag-list.tag-list--add
            .ui-tag__wrapper(
              @click="tagMenuShow = !tagMenuShow"
            )
              .ui-tag
                span.tag-title Add new
                span.button-close(
                  :class="{active: !tagMenuShow}"
                )
          transition(name="fade")
            .tag-list.tag-list-menu(
              v-if="tagMenuShow"
            )
              input.tag-add--input(
                type="text"
                placeholder="New tag"
                v-model="tagTitle"
                @keyup.enter="newTag"
              )
              .button.button-default(
                @click="newTag"
              ) Send
          .tag-list
            transition-group(
              enter-active-class="animated fadeInRight"
              leave-active-class="animated fadeOutDown"
            )
              .ui-tag__wrapper(
                v-for="tag in tags"
                :key="tag.title"
              )
                .ui-tag(
                  @click="addTag(tag)"
                  :class="{used: tag.use}"
                )
                  span.tag-title {{ tag.title }}
                  span.button-close
          .button-list
              button.button.button--round.button-primary(
                type="submit"
                :disabled="submitStatus === 'PENDING'"
              ) Send
</template>

<script>
import { required } from 'vuelidate/lib/validators'
export default {
  data () {
    return {
      submitStatus: null,
      taskTitle: '',
      taskDescription: '',
      whatWatch: 'movie',
      // Time
      movieHours: '1',
      movieMinutes: '40',
      seriesSeasons: '5',
      seriesEpisodes: '10',
      seriesEpisodeLength: '45',
      // Tags
      tagTitle: '',
      usedTags: [],
      tagMenuShow: false
    }
  },
  validations: {
    taskTitle: {
      required
    }
  },
  methods: {
    newTag () {
      if (this.tagTitle === '') {
        return
      }
      const tag = {
        title: this.tagTitle,
        use: false
      }
      this.$store.dispatch('newTag', tag)
    },
    onSubmit () {
      this.$v.$touch()

      if (this.$v.$invalid) {
        console.log('ERROR')
        this.submitStatus = 'ERROR'
      } else {
        // Vaild
        console.log('SEND')
        this.submitStatus = 'PENDING'
        // Firebase waiting
        setTimeout(() => {
          this.submitStatus = 'OK'
        }, 500)

        let time
        if (this.whatWatch === 'movie') {
          time = this.movieTime
        } else {
          time = this.seriesTime
        }
        const task = {
          title: this.taskTitle,
          description: this.taskDescription,
          whatWatch: this.whatWatch,
          time,
          tags: this.usedTags,
          completed: false,
          editing: false
        }

        this.$store.dispatch('newTask', task)
        console.log(task)
        // reset
        this.taskTitle = ''
        this.taskDescription = ''

        this.$v.$reset()

        this.tagMenuShow = false
        this.tagTitle = ''
        this.usedTags = []

        for (let i = 0; i < this.tags.length; i++) {
          this.tags[i].use = false
        }
      }
    },
    getTotalTime (minutes) {
      let hours = Math.trunc(minutes / 60)
      let min = minutes % 60
      return hours + ' Hours, ' + min + ' Minutes'
    },
    addTag (tag) {
      tag.use = !tag.use
      if (tag.use) {
        this.usedTags.push({
          title: tag.title
        })
      } else {
        this.usedTags.splice(tag.title, 1)
      }
    }
  },
  computed: {
    tags () {
      return this.$store.getters.tags
    },

    movieTime () {
      let min = (this.movieHours * 60) + (this.movieMinutes * 1)
      return this.getTotalTime(min)
    },
    seriesTime () {
      let min = this.seriesSeasons * this.seriesEpisodes * this.seriesEpisodeLength
      return this.getTotalTime(min)
    }
  }
}
</script>

<style lang="stylus" scoped>
.option-list
  display flex
  align-items center
  margin-bottom 20px
  .what-watch--radio
    margin-right 12px
  input
    margin-bottom 0
  label
    margin-right 20px
    margin-bottom 0
    &:last-child
      margin-right 0

.total-time
  margin-bottom 20px

.time-title
  display block
  margin-bottom 6px

.time-input
  max-width 80px
  margin-right 10px

.tag-list
  margin-bottom 20px

.ui-tag__wrapper
  margin-right 10px
  margin-bottom 10px
  &:last-child
    margin-right 0

.ui-tag
  .button-close
    &.active
      transform rotate(45deg)
  &.used
    background-color: #444ce0
    color #fff
    .button-close
      &:before,
      &:after
        background-color: #fff
.tag-list-menu
  display flex
  justify-content space-between
  align-items center

.tag-add--input
  margin-bottom 0
  margin-right 10px
  height 42px

.button-list
  display flex
  justify-content flex-end

.form-item
  .error
    display none
    margin-bottom 8px
    font-size 13.4px
    color #fc5c65
  &.errorInput
    .error
      display block
input
  &.error
    border-color #fc5c65
</style>
