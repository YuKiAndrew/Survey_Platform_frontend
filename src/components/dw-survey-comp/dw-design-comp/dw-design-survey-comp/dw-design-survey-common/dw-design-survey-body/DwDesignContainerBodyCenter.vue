<template>
  <div class="dw-container-body-center">
    <div style="min-height: 600px;">
      <div v-if="survey.hasOwnProperty('surveyAttrs') && survey.surveyAttrs.hasOwnProperty('scoreAttr') && survey.surveyAttrs.scoreAttr.enabled" style="font-size: 12px;color: #014ab6;padding: 20px 20px 0 20px;">
        本卷最大总分：<strong style="">{{ survey.surveyAttrs.scoreAttr.maxScore }}</strong> 分
      </div>
      <div style="text-align: center;padding: 20px;">
        <dw-text-edit-label-common v-model="survey.surveyNameObj" :survey="survey" ></dw-text-edit-label-common>
      </div>
      <div style="padding: 0 40px;">
        <dw-text-edit-label-common v-model="survey.surveyDetail.surveyNodeObj" :survey="survey" ></dw-text-edit-label-common>
      </div>
      <div style="margin-top: 10px;">
        <div>
          <!--          v-show="survey.questions.length<=0"-->
          <div key="dws_xxx" style="text-align: center;font-size: 13px;color: grey;line-height: 30px;">
            拖动上方工具栏题目控件到下方区域，可以增加新题目
          </div>
          <draggable
            v-model="survey.questions"
            :force-fallback="true"
            :group="{ name: 'questionGroup', pull: false, put: true }"
            handle=".dwMoveSortQu"
            animation="300"
            drag-class="dwDragClass"
            ghost-class="dwGhostClass"
            chosen-class="dwChosenClass"
            class="dwDraggable"
            @add="onAdd"
            @start="onStart"
            @end="onEnd">
            <transition-group>
              <div v-for="(item, index) in survey.questions" :key="item.hasOwnProperty('dwId') ? `quDwId_${item.dwId}`: `Qu_${index}`" >
                <dw-design-question ref="designQuestion" :index="index" :item="item" v-model="survey" ></dw-design-question>
              </div>
            </transition-group>
          </draggable>
          <div style="text-align: center;padding-top: 20px;padding-bottom: 40px;">
            <el-button type="primary" size="small" plain @click="addNewQu"><i class="fa-solid fa-plus"></i> 增加新题</el-button>
            <!--            <el-button size="small"><i class="fa-solid fa-layer-group"></i> 批量增加</el-button>-->
          </div>
        </div>
      </div>
    </div>
    <!--    <div style=""><dw-footer></dw-footer></div>-->
    <dw-add-new-qu-dialog ref="dwAddNewQuDialog" v-model="survey"></dw-add-new-qu-dialog>
  </div>
</template>

<script>
import DwDesignToolbar from '../../dw-design-survey-layouts/dw-tb-layout/dw-design-toolbar/DwDesignToolbar.vue'
import DwDesignHeader from '../../dw-design-survey-layouts/dw-tb-layout/comp/DwDesignHeader'
import DwTextEditLabelCommon from '../DwTextEditLabelCommon'
import DwDesignQuRadio from '../../dw-design-survey-question/dw-design-questions/dw-design-qu-radio/DwDesignQuRadio'
import DwTextEditLabel from '../DwTextEditLabel'
import DwDesignQuestion from '../../dw-design-survey-question/DwDesignQuestion'
import draggable from 'vuedraggable'
import DwFooter from '../../../../../layouts/DwFooter'
import DwDesignQuestionCommon from '../../dw-design-survey-question/DwDesignQuestionCommon.vue'
import {dwResetQuestionRefreshValue} from '../../../../dw-utils/dw-survey-update-question'
import {resetQuestion} from '../../../../dw-utils/dw-survey-parse'
import DwAddNewQuDialog
  from '../../dw-design-survey-layouts/dw-tb-layout/dw-design-toolbar/components/DwAddNewQuDialog.vue'
import {caleDesignSurveySumScore} from '../../../../dw-utils/dw-common/dw-survey-design-score'

export default {
  name: 'DwDesignContainerBodyCenter',
  components: {
    DwAddNewQuDialog,
    DwDesignQuestionCommon,
    DwFooter,
    DwDesignToolbar,
    DwDesignHeader,
    DwTextEditLabelCommon,
    DwDesignQuRadio,
    DwTextEditLabel,
    DwDesignQuestion,
    draggable
  },
  model: {
    prop: 'survey',
    event: 'update-survey'
  },
  props: {
    survey: {type: Object, default: () => {}}
  },
  data () {
    return {
      drag: false
    }
  },
  mounted () {
    this.handleResize()
    window.addEventListener('resize', this.handleResize, true)
  },
  beforeDestroy () {
    // 记得在组件销毁前移除事件监听器，以避免潜在的内存泄漏。 考虑提到全局
    window.removeEventListener('resize', this.handleResize)
  },
  methods: {
    onAdd (attrs) {
      console.debug('onAdd attrs', attrs)
      // this.documentClick()
      // 自动执行focus事件
      const newIndex = attrs.newIndex
      console.debug('newIndex', newIndex)
      this.refreshData(newIndex)
      // this.survey.questions[newIndex].quTitleObj.isNew = true // 新加入的不需要通过这个获得焦点，如果下面题目标记isNew 然后再初始化进行执行
      this.survey.questions[newIndex].isNew = true
      // 还没选项的ID，不管有没有，全部需要重置
      resetQuestion(this.survey.questions[newIndex])
      // 如果已经配置了分值则自动计算分数并显示
      caleDesignSurveySumScore(this.survey, newIndex)
    },
    onStart () {
      this.drag = true
      this.$emit('start-drag-container')
    },
    onEnd (attrs) {
      this.$emit('end-drag')
      console.debug('attrs', attrs)
      this.drag = false
      // 更新状态
      const newIndex = attrs.newIndex
      const oldIndex = attrs.oldIndex
      if (newIndex>oldIndex) {
        this.refreshData(oldIndex)
      } else {
        this.refreshData(newIndex)
      }
      this.documentClick()
    },
    documentClick () {
      /*
      const curObjs = this.survey.curEditObj
      for (let i = 0; i < curObjs.length; i++) {
        this.survey.curEditObj[i].itemClick = false
      }
      */
    },
    refreshQuData (data) {
      // 关于状态的刷新，除通过方法调用硬刷新，还可以通过选项数据上绑定的itemClick实现
      this.$nextTick(() => {
        const quCommonItems = this.$refs.designQuestion
        for (let i=0; i<quCommonItems.length; i++) quCommonItems[i].refreshQuData(data)
      })
    },
    refreshData (quIndex) {
      const questions = this.survey.questions
      questions.forEach((item, index) => {
        if (index>=quIndex) {
          this.survey.questions.splice(index, 1, dwResetQuestionRefreshValue(JSON.parse(JSON.stringify(item))))
          /*
          const question = dwResetQuestionRefreshValue(JSON.parse(JSON.stringify(item)))
          if (index===quIndex) question.quTitleObj.isNew = true
          this.survey.questions.splice(index, 1, question)*/
        }
      })
    },
    addNewQu () {
      this.$refs.dwAddNewQuDialog.openDialog()
    },
    handleResize () {
      const windowWidth = window.innerWidth
      if (this.survey.hasOwnProperty('clientBrowser')) {
        this.survey.clientBrowser.windowWidth = windowWidth
        this.survey.clientBrowser.matrixWidth = windowWidth*0.66 - 130
      }
    }
  }
}
</script>

<style scoped>
.dw-container-body-center{
  position: relative;
  z-index: 50;
  width: 100%;
  border:1px solid #f3f3f3;
  min-height: 600px;
}
.dwDraggable span{
  width: 100%;
  min-height: 300px;
  display: inline-block;
  background-color: #f7f7f7;
}
</style>
