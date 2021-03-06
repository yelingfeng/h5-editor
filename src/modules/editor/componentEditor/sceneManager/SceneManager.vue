<template>
  <div class="gls-scene-manager">
    <!--头部-->
    <div class="manager-header">
      <a>场景编辑</a>
      <gls-button type="black" hoverType="hover-success" v-b-tooltip.hover.left="'复制当前场景'" @click.native="copyScene()">
        <div class="iconfont icon-copy"></div>
      </gls-button>
      <gls-button type="black" hoverType="hover-danger" @click.native="showDeleteSceneDialog()" v-b-tooltip.hover.left="'删除当前场景'">
        <div class="iconfont icon-delete"></div>
      </gls-button>
    </div>

    <!--每个场景的按钮-->
    <div class="manager-content" ref="manageContent">
      <draggable v-model="project.scenes" @end="onEnd($event)">
        <transition-group>
          <div :class="{scene:true,selected:project.selectedScene === scene}" v-for="(scene,index) in project.scenes"
               @click="changeSelectedScene(scene)" :key="scene.props.id">
            <span class="number"></span>
            <span class="index">{{index}}</span>
            <gls-scene-name-edit @dblclick.native="showInput($event)" ref="nameEdit" :scene="scene"
                                 class="name"></gls-scene-name-edit>
          </div>
        </transition-group>
      </draggable>
    </div>

    <!--新增场景的按钮-->
    <div class="gls-scene-manager-add-scene" @click="addScene()">
      <span class="iconfont icon-add"></span>
    </div>

    <!--删除页面的询问窗口-->
    <b-modal ref="deleteSceneConfirm" hide-footer hide-header-close>
      <div class="d-block text-center">
        <h6>页面删除后无法还原</h6>
      </div>
      <div class="text-center">
        <b-btn class="m-3" variant="danger sm" :style="{color:'#FFFFFF'}" @click="deleteScene">坚持删除</b-btn>
        <b-btn class="m-3" variant="success sm" :style="{color:'#FFFFFF'}" @click="hideDeleteSceneDialog">稍后决定</b-btn>
      </div>
    </b-modal>
  </div>
</template>

<script lang="ts">
  /**
   * create by 给力叔 2017/9/16
   * 场景管理组件
   */
  import Vue from 'vue'
  import {Component, Inject, Model, Prop, Watch} from 'vue-property-decorator'
  import glsButton from '../../common/button/Button.vue';
  import {Project} from "../../core/project/Project";
  import {Scene} from "../../core/scene/Scene";
  import {SceneService} from "../../core/factorys/scsne/SceneService";
  import GlsSceneNameEdit from './SceneNameEdit.vue'
  import {Renderer} from "../../../../common/render/Renderer";
  import {Draggable} from "../../common/interact/Draggable";
  import {IDraggableResult} from "../../common/dragable/IDraggableResult";
  import {Rect} from "../../core/geom/Rect";
  import draggable from 'vuedraggable'
  import {ProjectService} from "../../core/project/ProjectService";

  @Component({
    name: "gls-scene-manager",
    components: {
      glsButton: glsButton,
      GlsSceneNameEdit: GlsSceneNameEdit,
      draggable: draggable
    }
  })
  export default class SceneManager
    extends Vue {
    @Prop({required: true}) project: Project;

    _documentClick: Function;

    mounted() {
      //点击其他位置，就隐藏正在显示的名称输入框
      this._documentClick = Renderer.addEventListener(document, "click", (event) => {
        let current = event.target;
        let map: Map<any, boolean> = new Map();
        do {
          map.set(current, true);
          current = current.parentNode;
        } while (current.parentNode);
        (<any>this.$refs.nameEdit).forEach((nameEdit) => {
          if (!map.has(nameEdit.$el)) {
            nameEdit.showInput = false;
          }
        })
      })
    }

    addScene() {
      SceneService.getInstance().addScene(this.project);
    }

    copyScene(){
      SceneService.getInstance().copyScene(this.project);
    }

    showDeleteSceneDialog() {
      if (this.project.selectedScene) {
        (<any>this.$refs.deleteSceneConfirm).show();
      }
    }

    deleteScene() {
      SceneService.getInstance().deleteScene(this.project);
      this.hideDeleteSceneDialog();
    }

    hideDeleteSceneDialog() {
      (<any>this.$refs.deleteSceneConfirm).hide();
    }

    /**
     * 双击了名称，就显示名称的输入框
     * @param $event
     */
    showInput($event) {
      (<any>this.$refs.nameEdit).forEach((nameEdit) => {
        if ($event.currentTarget == nameEdit.$el) {
          nameEdit.showInput = true;
        } else {
          nameEdit.showInput = false;
        }
      })
    }

    changeSelectedScene(scene:Scene){
      this.project.selectedScene = scene;
    }

    onEnd($event){
      SceneService.getInstance().changeScene($event.oldIndex,$event.newIndex);
    }

    destroyed() {
      this._documentClick && this._documentClick();
    }
  }
</script>

<style scoped lang="scss">
  @import "../../common/scss/index.scss";

  $width: 260px;
  $add-button-height: 50px;
  .gls-scene-manager {
    position: fixed;
    top: $header_height;
    right: 0;
    bottom: 0;
    width: $width;
    background-color: #FFFFFF;
  }

  $manager_header_height: 45px;

  .manager-header {
    height: $manager_header_height;
    background-color: #76838F;
    width: 100%;
    padding: 5px;
    box-sizing: border-box;
    position: relative;
    color: white;
    a {
      line-height: $manager_header_height - 10px;
    }
    /*两个按钮向右浮动*/
    .gls-button {
      float: right;
    }
    /*按钮之间增加间距*/
    .delete-button {
      margin-right: 3px;
    }
  }

  $scene_button_height: 70px;

  .manager-content {
    position: fixed;
    top: $header_height + $manager_header_height;
    right: 0;
    bottom: $add-button-height;
    width: $width;
    overflow-y: auto;
    overflow-x: hidden;

    .yellow-clone {
      height: $scene_button_height;
      background-color: #FFFA90;
    }
  }

  /*每个场景的选项*/
  .scene {
    height: $scene_button_height;
    font-size: 12px;
    margin: 5px 5px 0 5px;
    position: relative;
    border: 1px solid #ccd5db;
    color: #76838f;
    cursor: pointer;
    background-color: #FFFFFF;
    width: $width;

    &.selected {
      .number {
        border-top-color: #08a1ef;
      }
    }

    &:hover {
      background-color: #F0F3F4;
    }

    .number {
      border-top: 35px solid #76838F;
      border-right: 35px solid transparent;
      position: relative;
      top: 0px;
    }

    .index {
      position: absolute;
      top: 0px;
      left: 0px;
      color: #FFFFFF;
      display: inline-block;
      width: 20px;
      text-align: center;
    }

    .name {
      position: absolute;
      top: 25px;
      left: 35px;
    }
  }

  .gls-button.hover-danger {
    margin-right: 3px;
  }

  .gls-scene-manager-add-scene {
    width: $width;
    height: $add-button-height;
    text-align: center;
    position: fixed;
    bottom: 0;
    right: 0;
    background-color: #31364A;
    cursor: pointer;
    line-height: 48px;
    color: #FFFFFF;

    .iconfont {
      font-size: 48px;
      top: 16px;
      position: relative;
    }
  }
</style>
