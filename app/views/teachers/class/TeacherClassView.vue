<template>
    <loading-progress :loading="loading" :progress="progress">
        <breadcrumbs v-if="!loading" :links="links"></breadcrumbs>
        <backbone-view v-if="!loading" :backbone-view="backboneViewInstance"></backbone-view>
    </loading-progress>
</template>

<script>
  import { mapActions, mapState } from 'vuex'
  import TeacherClassView from 'app/views/courses/TeacherClassView'
  import LoadingProgress from '../../core/LoadingProgress'
  import Breadcrumbs from '../../common/BreadcrumbComponent'

  const BackboneView = {
    props: {
      backboneView: Object
    },

    render: function () {},

    mounted() {
      this.$props.backboneView.render()
      this.$el.parentElement.appendChild(this.$props.backboneView.el)
    },
  }

  export default {
    components: {
      LoadingProgress,
      BackboneView,
      Breadcrumbs
    },

    data: function () {
      return {
        loading: false,
        progress: 0,

        backboneViewInstance: new TeacherClassView(
          { vue: true, readOnly: true },
          this.$route.params.classroomId
        ),
        links: [{
          href: '/school-administrator',
          i18n: 'school_administrator.my_teachers'
        }, {
          href: `/school-administrator/teachers/${this.$route.teacherId}`,
          text: this.teacherName
        }, {
          text: this.classroom.name
        }]
      }
    },

    methods: Object.assign({},
      mapActions({
        fetchClassroomForId: 'classrooms/fetchClassroomsForId'
      }),
      {
        showLoading: function () {
          this.loading = true
        },

        hideLoading: function () {
          this.loading = false
        },

        updateLoadingProgress: function (progress) {
          this.progress = progress
        },

        initialize: function () {
          this.fetchClassroomForId(this.$route.params.classroomId)
          this.fetchTeacherForId(this.$route.params.teacherId)
        }
      }
    ),

    computed: Object.assign({},
      mapState('classrooms', {
        // TODO: Improve loading in this file
        // classroomLoading: function (state) {
        //   return state.loading.byClassroom(this.$route.params.classroomId)
        // },
        classroom: function (state) {
          return state.classrooms.byClassroom(this.$route.params.classroomId)
        },
      }),
      // mapState('schoolAdministrator', {
      //   teacherLoading: function (state) {
      //     return state.loading.byTeacher(this.$route.params.teacherId)
      //   },
      //   teacherForId: function (state) {
      //     return state.teacher
      //   }
      // })
    ),

    created() {
      this.backboneViewInstance.on('loading:show', this.showLoading)
      this.backboneViewInstance.on('loading:hide', this.hideLoading)
      this.backboneViewInstance.on('loading:progress', this.updateLoadingProgress)
      this.initialize()
    },

    updated() {
      this.initialize()
    },

    beforeDestroy() {
      this.backboneViewInstance.destroy()
    }
  }
</script>
