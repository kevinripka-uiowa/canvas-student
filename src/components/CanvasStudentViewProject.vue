<template>
  <div class="canvas-project-details">
    <template>

    <div id="project-details">

      <div class="flex">
        <div class="flex flex-center-both project-name">
          <div>
            <h1 class="serif fs-b1 color-w ta-center">{{project.project.name}}</h1>
            <template v-if="project.project.type == 'CustomProject'">
              <p class="color-w m-0 ta-center">Shared by {{project.project.teacher.name}}</p>
              <p class="color-w m-0 ta-center">{{project.project.teacher.school}}</p>
            </template>
          </div>
        </div>

        <template v-if="project.project.type == 'Project'">
        <div class="project-video">
          <div class="videoWrapper">
            <video controls class="videoWrapper__video" :poster="project.intro_video_poster" preload="auto">
              <template v-for="url in project.intro_video_urls_filtered">
                <source :key="url" :src="url">
              </template>
            </video>
          </div>
        </div>
        </template>

        <template v-if="project.project.type == 'CustomProject'">
          <div class="project-video">
            <div class="videoWrapper">
              <img :src="project.project.image" v-if="project.project.image_or_video=='image'">
              <video :src="project.project.video" v-else-if="project.project.image_or_video='video'" controls></video>
            </div>
          </div>
        </template>

      </div>


      <div class="flex flex-jc-sb flex-reverse project-details">

        <div class="flex-col">
          <div class="frame p-base">
            <h3 class="color-p fs-base serif w-700 m-0-0-s4">Description</h3>

                <vue-markdown :source="project.description || project.project.description"></vue-markdown>

          </div>

          <div class="frame p-base">
            <h3 class="color-p fs-base serif w-700 m-0-0-s4">Project Details</h3>
            <table class="canvas-table">
            <tr v-if="project.project.outdoors || project.project.indoors || project.project.location_type == 'ON'">
              <th scope="row">Spend the time</th>
              <td v-if="project.project.location_type == 'ON'">Online</td>
              <td v-else-if="project.project.outdoors && project.project.indoors">Indoors and Outdoors</td>
              <td v-else-if="project.project.outdoors">Outdoors</td>
              <td v-else>Indoors</td>
            </tr>
            <tr v-if="project.project.topics.length > 0">
              <th scope="row">Topics</th>
              <td>{{topics}}</td>
            </tr>
            <tr v-if="project.project.activities.length > 0">
              <th scope="row">Type of Activity</th>
              <td>{{activities}}</td>
            </tr>
            <tr v-if="project.project.classroom_materials">
              <th scope="row">Classroom materials</th>
              <td>{{ project.project.classroom_materials }}</td>
            </tr>
          </table>
          </div>

          <div v-if="project.student_resource_1 && project.student_resource_1.url !== ''" class="frame p-base">
            <h3 class="color-p fs-base serif w-700 m-0-0-s4">Additional Resources for Students</h3>
            <ul class="standard">
              <li><a :href="project.student_resource_1.url" target="_blank">{{project.student_resource_1.name || project.student_resource_1.url}}</a></li>
              <li v-if="!!project.student_resource_2.url"><a :href="project.student_resource_2.url" target="_blank">{{project.student_resource_2.name || project.student_resource_2.url}}</a></li>
              <li v-if="!!project.student_resource_3.url"><a :href="project.student_resource_2.url" target="_blank">{{project.student_resource_3.name || project.student_resource_3.url}}</a></li>
              <li v-if="!!project.student_resource_4.url"><a :href="project.student_resource_4.url" target="_blank">{{project.student_resource_4.name || project.student_resource_4.url}}</a></li>
            </ul>
          </div>
        </div><!-- end .flex-col -->

        <div class="flex-col">

          <div class="frame p-base div1">
            <h3 class="color-p fs-base serif w-700 m-0-0-s4">Time Required</h3>
            <p class="serif fs-b1 color-b">{{project.time_required || project.project.time }}</p>
          </div>

          <div class="frame p-base div2">
            <h3 class="color-p fs-base serif w-700 m-0-0-s4">Materials Needed</h3>

              <template v-if="project.project.type == 'Project'">
                <ul class="standard">
                  <li v-for="(mat,i) in project.materials" :key="'mat'+i">{{ mat }}</li>
                </ul>
              </template>
              <template v-if="project.project.type == 'CustomProject'">
                <p>{{project.project.materials}}</p>
              </template>

          </div>
        </div><!-- end .flex-col -->


      </div><!-- end .grid -->

      <div class="frame p-base m-0-basehalf">
          <h3 class="color-p fs-base serif w-700 m-0-0-s4">Project Instructions</h3>
          <ol class="instructions">
            <template v-if="project.project.type == 'Project'">
              <li v-for="(step, index) in project.instruction_steps_active" :key="'step' + index">{{step}}</li>
            </template>
            <template v-if="project.project.type == 'CustomProject'">
              <div class="m-0-0-base">
              <vue-markdown :source="project.project.steps"></vue-markdown>
            </div>
            </template>
        </ol>

        <a @click="triggerDoProjectTab" class="cbtn-primary">Do Project!</a>

      </div>



    </div><!-- end #project-details -->

  </template>




    </div>
</template>

<script>
import VueMarkdown from 'vue-markdown'
export default {
  name: 'StudentViewProject',
  props: ['project'],
  data: function(){
    return {
      questions: null
    }
  },
  components: {
      VueMarkdown
  },
  methods: {
    scrollToTop(){
      window.scrollTo(0,0)
    },
    triggerDoProjectTab(){
      this.$emit('triggerTab',2);
    }

  },
  computed: {
    topics(){
      let topics = []
      this.project.project.topics.forEach(d => {
        topics.push(d.label)
      })
      return topics.join(', ')
    },
    activities(){
      let act = []
      this.project.project.activities.forEach(d => {
        act.push(d.label)
      })
      return act.join(', ')
    }

  },
  mounted(){
    this.scrollToTop()


  },
  created(){
    let ctx = this

    // filter videos so no nulls
    ctx.project.intro_video_urls_filtered = [];
    ctx.project.intro_video_urls.forEach(function(v){
      if(v){ctx.project.intro_video_urls_filtered.push(v)}
    })
  }
}
</script>

<style lang="scss">
@import '../assets/css/CanvasVariables.scss';
@import '../assets/css/CanvasProject.scss';
</style>
