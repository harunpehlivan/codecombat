<template>
    <loading-progress :loading="loading" :progress="progress">
        <breadcrumbs v-if="!loading" v-bind:links="links"></breadcrumbs>
        <backbone-view v-if="!loading" :backbone-view="backboneViewInstance"></backbone-view>
    </loading-progress>
</template>

<script>
  import { mapState } from 'vuex'
  import TeacherClassView from 'app/views/courses/TeacherClassView'
  import LoadingProgress from '../../core/LoadingProgress'
  import Breadcrumbs from '../../common/BreadcrumbComponent'
  import Classroom from 'models/Classroom'

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
          text: this.classroomName()
        }]
      }
    },

    methods: {
      showLoading: function () {
        this.loading = true
      },

      hideLoading: function () {
        this.loading = false
      },

      updateLoadingProgress: function (progress) {
        this.progress = progress
      },

      classroomName: function () {
        console.log('trying to get classroomName:')
        console.log(this.$route.params.classroomId)
        const classroom = new Classroom(this.$route.params.classroomId)
        console.log(classroom)
        console.log(classroom.get('name'))
        return classroom.get('name')
      }
    },

    computed: Object.assign({},
      mapState('schoolAdministrator', [
        'administratedTeachers'
      ]),

      {
        teacherName: () => {
          console.log('inside teacherName now!')
          console.log(this.administratedTeachers)
          const teacher = this.administratedTeachers.find(t => t.id === this.$route.teacherId)
          console.log(teacher)
          return teacher.firstName ? `${teacher.firstName} ${teacher.lastName}` : teacher.name
        }
      }
    ),


    created() {
      this.backboneViewInstance.on('loading:show', this.showLoading)
      this.backboneViewInstance.on('loading:hide', this.hideLoading)
      this.backboneViewInstance.on('loading:progress', this.updateLoadingProgress)
    },

    beforeDestroy() {
      this.backboneViewInstance.destroy()
    }
  }
</script>
