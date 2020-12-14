<template>
  <div class='settings' :class='{collapsed: settingsPanel.collapsed}'>
    <div class='block network-node'  v-if='networkNode'>
      <div class='title'>Network node Play<a class='reset-all' :class='{"syntax-visible": syntaxHelpVisible}' href='#' @click.prevent='syntaxHelpVisible = !syntaxHelpVisible' title='click to learn more about syntax'>语法帮助</a></div>
      <syntax v-if='syntaxHelpVisible' @close='syntaxHelpVisible = false'></syntax>
      <code-editor :model='networkNode'></code-editor>
    </div>
    <div class='block' v-if='showBindings'>
      <Inputs :vm='inputsModel'></Inputs>
	  
    </div>
    <form class='block' @submit.prevent='onSubmit'>
      <div class='title'>设置<a class='reset-all' href='?' title='set default settings'>恢复默认</a> </div>
	  
      <div class='row'>
        <div class='col'>节点颜色吧</div>
        <div class='col'> 
          <select v-model='selectedColorMode' @change='changeColor'>
              <option value='1'>red</option>
              <option value='2'>yellow</option>
              <option value='3'>green</option>
	        </select>
        </div>
        <help-icon @show='selectedColorHelp = !selectedColorHelp' :class='{open: selectedColorHelp}'></help-icon>
      </div>
	  
      <div class='row help' v-if='selectedColorHelp'>
        <div>
          <p>可以设置网络中节点颜色</p>
          <p>默认选项为"red"</p>
        </div>
      </div>
	  
	  <div class='row'>
        <div class='col'>生成网络类型</div>
        <div class='col'> 
          <select v-model='selectedGeneratingMode' @change='selectedGeneratingMode'>
              <option value='1'>ER</option>
              <option value='2'>WS</option>
              <option value='3'>BA</option>
	        </select>
        </div>
        <help-icon @show='selectedGeneratingModeHelp = !selectedGeneratingModeHelp' :class='{open: selectedGeneratingMode}'></help-icon>
      </div>
	  
      <div class='row help' v-if='selectedGeneratingModeHelp'>
        <div>
          <p>定义网络类型。不同的网络类型代表不同网络的生成方式。我们的展示包括三种网络类型：ER网络，WS网络，BA网络。</p>
		  
          <ul>
		  
           <li><i>ER 网络</i>该模型的基本思想是以概率P连接N个节点中的每对节点，更确切地说，给定网络节点总数N，网络中任意两个节点以概率P连接，所有生成的网络记录为G（N，P），形成一个概率空间。由于网络中的连接数是一个随机变量x，其值可以从0到n*（n-1）/2，因此生成的不同网络的总数服从二项分布。</li>
           
		       <li><i>WS 网络</i>是小世界网络的基本模型，与ER网络模型相比，该模型能更好地描述实际网络的大聚类和短平均路径距离等特点。
			         它的实现步骤是：首先，给定一个正则网络：如果我们有一个一维有限正则网络有N个节点，每个节点及其最近邻节点K=2i，通常为1≤K≤N，则重写旧连接：每个概率为p的旧连接都被重路由，其方法是将连接的一个端点随机放置在一个新的位置，但除了自连接和重复连接外，还需要进行排列。因为不允许重复连接，所以在给定的常规网络中只有N*K/2个连接。重新布线时，新位置随机放置在每个旧连接的选定端点上。因此，重写的连接数为p*N*K/2。由于随机性，这些重写的行可能有长距离的连接，这被称为快捷方式。显然，当p=0时，它仍然是一个给定的正则网，当p=1时，我们将得到一个特殊的随机网。随着P的增加，我们可以看到从规则网到随机网的变化。</li>  
		       <li><i>BA 网络</i>BA模型的算法：首先，给定n0节点，在每个时间步重复添加一个新节点和m（m≤n0）个新连接。新节点根据优先概率ki/∑kJ选择旧节点i与其连接，其中ki是旧节点i的度。BA网络使用随机数图.barabasi\albert_u-Graph（n，m）方法生成一个每次有n个节点和m个边的BA无标度网络。</li>
          </ul>
          <p>默认选项是"ER"</p>
        </div>
      </div>
	  
	    <div class='row'>
        <div class='col'>失效策略</div>
        <div class='col'> 
          <select v-model='selectedMode' @change='selectedMode'>
              <option value='1'>度中心性</option>
              <option value='2'>介数中心性</option>
              <option value='3'>接近中心性</option>
			        <option value='4'>特征向量中心性</option>
              <option value='4'>Katz中心性</option>
              <option value='4'>PageRank</option>
	        </select>
        </div>
        <help-icon @show='selectedModeHelp = !selectedModeHelp' :class='{open: selectedModeHelp}'></help-icon>
      </div>
	  
      <div class='row help' v-if='selectedModeHelp'>
        <div>
          <p>该选项可指定现有网络的失效策略</p>
          <ul>
            <li><i>度中心性</i> 一个节点与其他节点相连越多，它的度越大。度大的节点优先失效，形成新图，重复此过程。</li>
            <li><i>介数中心性</i>  </li>
            <li><i>接近中心性</i> highlights zones based on velocity vector angle.</li>
			      <li><i>特征向量中心性</i> highlights zones based on velocity vector angle.</li>
            <li><i>Katz中心性</i> highlights zones based on velocity vector angle.</li>
            <li><i>PageRank</i> highlights zones based on velocity vector angle.</li>
          </ul>
          <p>默认失效策略为“度中心性失效”</p>
        </div>
      </div>
	  
      <div class='row' v-if='soundAvailable'>
        <div class='col'>SoundCloud track</div>
        <div class='col'>
          <input type='text' v-model='soundCloudLink'>
          <a href='#' @click.prevent='loadSound'>load</a>
        </div>
      </div>
      <div class='row' v-if='soundAvailable'>
        <audio ref='player' controls='' autoplay='' preload autobuffer></audio>
      </div>
	  
      <div class='row'>
        <div class='col'>节点数量</div>
        <div class='col'><input type='number' :step='particleCountDelta' v-model='particlesCount' @keyup.enter='onSubmit' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></div>
        <help-icon @show='particleCountHelpVisible = !particleCountHelpVisible' :class='{open: particleCountHelpVisible}'></help-icon>
      </div>
      <div class='row help' v-if='particleCountHelpVisible'>
        <div>
          <p>数值越大，生成的网络越大。</p>
          <p>推荐的数值在<b>10,000</b>和<b>100,000</b>之间</p>
        </div>
      </div>
	  
      <div class='row'>
        <div class='col'>网络演化速度 </div>
        <div class='col'><input type='number' :step='fadeoutDelta'  v-model='fadeOutSpeed' @keyup.enter='onSubmit' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></div>
        <help-icon @show='fadeoutDeltaHelp = !fadeoutDeltaHelp' :class='{open: fadeoutDeltaHelp}'></help-icon>
      </div>
      <div class='row help' v-if='fadeoutDeltaHelp'>
        <div>
          <p>值越大，生成网络速度越快。</p>
          <ul>
            <li>Setting this value to <b>1</b> will keep particle trace forever.</li>
            <li> Setting this value to <b>0</b> will leave no trace at all</li>
          </ul>
          <p>推荐速度是<b>0.998</b></p>
        </div>
      </div>
	  
      <div class='row'>
        <div class='col'>节点大小 </div>
        <div class='col'><input type='number' :step='resetProbabilityDelta'  v-model='dropProbability' @keyup.enter='onSubmit' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></div>
        <help-icon @show='resetProbabilityHelp = !resetProbabilityHelp' :class='{open: resetProbabilityHelp}'></help-icon>
      </div>
      <div class='row help' v-if='resetProbabilityHelp'>
        <div>
          <p>值越大，生成的节点越大。</p>
          <ul>
            <li>Setting this value to <b>1</b> will reset all particles on every frame. This can be a good option to "reset" an empty screen.</li>
            <li>Setting this value to <b>0</b> will prevent particles from jumping to a random spot. This can be a good option to trace particles trajectory.</li>
          </ul>
          <p>默认值是<b>0.009</b></p>
        </div>
      </div>
	  
      <div class='row'>
        <div class='col'>时间轴</div>
        <div class='col'><input type='number' :step='integrationStepDelta' v-model='timeStep' @keyup.enter='onSubmit' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false" ></div>
        <help-icon @show='integrationStepHelp = !integrationStepHelp' :class='{open: integrationStepHelp}'></help-icon>
      </div>
      <div class='row help' v-if='integrationStepHelp'>
        <div>
          <p>This parameter defines how fast time flies for each particle (or, to be more accurate, this is the integration step of the classical Runge-Kutta method)</p>
          <ul>
            <li>Increasing this value makes particles fly faster at risk of missing proper curve's turns.</li>
            <li>Making this value smaller increases the accuracy of particle's trajectory, and makes them move slower.</li>
          </ul>
          <p>Default value is <b>0.01</b></p>
        </div>
      </div>
	  
      <div class='bounding-box'>
        <div class='col title'>bounds</div>
        <div class='row'>
          <div class='col  center'><input type='number' v-model.lazy='minY' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></div>
        </div>
        <div class='row'>
          <div class='col min-x'><input type='number' v-model.lazy='minX' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></div>
          <a class='col reset' href='#' @click.prevent='goToOrigin' title='navigate to point (0,0)'>start</a>
          <div class='col max-x'><input type='number' v-model.lazy='maxX' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></div>
        </div>
        <div class='row center'>
          <div class='col center'><input type='number' v-model.lazy='maxY' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></div>
        </div>
      </div>
    </form>
  </div>
</template>
<script>
// TODO: This file becomes too big. Need to split.
import bus from '../lib/bus';
import isSmallScreen from '../lib/isSmallScreen';
import appState from '../lib/appState';
import SoundLoader from '../lib/sound/soundLoader';
import SoundCloudAudioSource from '../lib/sound/audioSource';
import config from '../lib/config';
import Syntax from './help/Syntax';
import HelpIcon from './help/Icon';
import CodeEditor from './CodeEditor';
import Inputs from './Inputs';

// Temporary disable this until API is finished.
const soundAvailable = config.isAudioEnabled;

export default {
  name: 'Settings',
  props: ['scene'],
  components: {
    Syntax,
    HelpIcon,
    CodeEditor,
    Inputs
  },
  mounted() {
    bus.on('scene-ready', this.onSceneReady, this);
    bus.on('bbox-change', this.updateBBox, this);

    if (soundAvailable) this.soundLoader = new SoundLoader(this.$refs.player);
  },
  beforeDestroy() {
    bus.off('scene-ready', this.onSceneReady, this);
    bus.off('bbox-change', this.updateBBox, this);
  },
  data() {
    return {
      soundCloudLink: 'https://soundcloud.com/mrfijiwiji/yours-truly',
      vectorField: null,
      settingsPanel: appState.settingsPanel,
      inputsModel: scene.inputsModel,
      showBindings: config.showBindings,
      particlesCount: 0,
      fadeOutSpeed: 0,
      dropProbability: 0,
      timeStep: 0,
      selectedColorMode: 0,
      soundAvailable: soundAvailable,
      // TODO: Need something better for help management?
      selectedColorHelp: false,
      selectedModeHelp: false,
      selectedGeneratingMode:false,
      selectedGeneratingModeHelp:false,
      syntaxHelpVisible: false,
      particleCountHelpVisible: false,
      fadeoutDeltaHelp: false,
      resetProbabilityHelp: false,
      integrationStepHelp: false,
      minX: 0, minY: 0,
      maxX: 0, maxY: 0
    };
  },
  watch: {
    'settingsPanel.collapsed': function(newValue) {
      bus.fire('settings-collapsed', newValue);
    },
    particlesCount(newValue, oldValue) {
      this.scene.setParticlesCount(parseInt(newValue, 10));
    },
    timeStep(newValue, oldValue) {
      this.scene.setIntegrationTimeStep(newValue);
    },
    fadeOutSpeed(newValue, oldValue) {
      this.scene.setFadeOutSpeed(newValue);
    },
    dropProbability(newValue, oldValue) {
      this.scene.setDropProbability(newValue);
    },
    selectedColorMode(newValue) {
      this.scene.setColorMode(newValue);
    },
    minX(newValue) { this.moveBoundingBox('minX', newValue) },
    maxX(newValue) { this.moveBoundingBox('maxX', newValue) },
    minY(newValue) { this.moveBoundingBox('minY', newValue) },
    maxY(newValue) { this.moveBoundingBox('maxY', newValue) },
  },
  computed: {
    particleCountDelta() {
      return exponentialStep(this.particlesCount);
    },
    integrationStepDelta() {
      var timeStep = this.timeStep;
      return exponentialStep(timeStep);
    },
    resetProbabilityDelta() {
      return exponentialStep(this.dropProbability);
    },
    fadeoutDelta() {
      var fadeOutSpeed = Number.parseFloat(this.fadeOutSpeed);

      var exp = Math.round(Math.log10(1 % fadeOutSpeed)) ;
      var dt = Math.pow(10, exp);
      if (dt + fadeOutSpeed >= 1) {
        dt /= 10;
      }
      return dt;
    }
  },
  methods: {
    moveBoundingBox(key, value) {
      if (this.ignoreBbox) {
        return;
      } 
      this.scene.moveBoundingBox({[key]: value});
    },
    loadSound() {
      if (!this.soundLoader) return;
      this.soundLoader.loadStream(this.soundCloudLink).then(e => {
        if (!this.audioSource) {
          this.audioSource = new SoundCloudAudioSource(this.$refs.player); 
        }
        this.audioSource.playStream(this.soundLoader.streamUrl())
      });
      // TODO: Error handling
    },
    goToOrigin() {
      this.scene.resetBoundingBox();
    },  
    onSubmit() {
      if (isSmallScreen()) {
        appState.settingsPanel.collapsed = true;
      }
    },
    changeColor(e) {
      this.selectedColorMode = e.target.value;
    },

    updateBackground(rgba) {
      this.scene.setBackgroundColor(rgba);
    },

    onSceneReady(scene) {
      this.vectorField = scene.vectorFieldEditorState;
      this.particlesCount = scene.getParticlesCount();
      this.fadeOutSpeed = scene.getFadeOutSpeed();
      this.dropProbability = scene.getDropProbability();
      this.timeStep = scene.getIntegrationTimeStep();
      this.selectedColorMode = scene.getColorMode();
      this.updateBBox();
    },

    updateBBox() {
      this.ignoreBbox = true;
      var bbox = scene.getBoundingBox();
      this.minX = bbox.minX;
      this.maxX = bbox.maxX;

      // Y is weird in my implementation. I know..
      this.minY = bbox.minY;
      this.maxY = bbox.maxY;
      if (this.prevBboxReset) clearTimeout(this.prevBboxReset);

      this.prevBboxReset = setTimeout(() => {
        this.ignoreBbox = false
        this.prevBboxReset = 0
      }, 50);
    },
  }
}

function exponentialStep(value) {
  var dt = Math.pow(10, Math.floor(Math.log10(value)));
  if (value - dt === 0) {
    // This is odd case when you are increasing number, but otherwise it's a good adjustment.
    return dt/10;
  }
  return dt;
}

function toColorString({r, g, b, a}) {
  if (a === 1.0) {
    return `#${hex(r)}${hex(g)}${hex(b)}`;
  }
  return `rgba(${r}, ${g}, ${b}, ${a})`;
}

function hex(x) {
  let value = x.toString(16).toUpperCase();
  if (value.length === 1) value = '0' + value;
  return value;
}
</script>

<style lang='stylus'>
@import "./shared.styl";
@import "./glsl-theme.styl";

help-background = #569CD6;
// 使用颜色范围有规定

.settings {
  color: secondary-text;
  left: 0;
  overflow-y: auto;
  border-top: 1px solid secondary-text;
  background: window-background;
  width: 100%;
  padding: 7px 7px 7px 7px;
}
.settings.collapsed {
  display: none;
}

.title {
  margin-bottom: 7px;
  color: primary-text;
  font-size: 18px;
}
.block {
  .col {
    align-items: center;
    display: flex;
  }
  .row {
    margin-top: 4px;
  }
  select {
    margin-left: 14px;
  }

  input[type='text'],
  input[type='number'] {
    background: transparent;
    color: primary-text;
    border: 1px solid transparent;
    padding: 7px;
    font-size: 16px;
    width: 100%;
    margin-left: 7px;
    &:focus {
      outline-offset: 0;
      outline: none;
      border: 1px dashed;
      background: #13294f;
    }
    &:invalid {
      box-shadow:none;
    }
  }
}

.help {
  margin: -7px;
  margin-bottom: 7px;
  padding: 7px 7px 14px 7px;
  background: help-background;
}
.title {
  a {
    float: right;
    font-size: 12px;
    font-style: italic;
    color: help-text-color;
    height: 30px;
    margin: -5px;
    padding: 7px;
  }

  a.syntax-visible {
    background: help-background;
    color: white;
    font-style: normal;
  }
}
form.block {
  margin-top: 12px;
  padding-top: 10px;
  display: flex;
  flex-direction: column;
}
.vector-field {
  pre.error {
    color: rgba(250, 232, 55, 1);
    overflow-y: auto;
  }
  pre.error.detail {
    overflow: none;
    white-space: normal;
    .hl {
      background-color: #172A4D;
      color: red;
      font-weight: bold;
    }
  }

  textarea {
    background: transparent;
    color: white;
    font-family: monospace;
    margin-top: 14px;
    padding: 0;
    padding-left: 14px;
    width: settings-width - 14px;
    font-size: 14px;
    border: 1px solid transparent;
    &:focus {
      outline: none;
      border: 1px dashed;
      background: #13294f;
    }
  }
}

.row {
  display: flex;
  flex-direction: row;
}

.center {
  justify-content: center;
}

audio {
  width: 100%;
}

.col {
  flex: 1;
}
a {
  text-decoration: none;
}

a.action {
  color: white;
  font-size: 16px;
}

a.help-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 25px;
  margin-right: -7px;
  svg {
    fill: secondary-text;
  }
  &.open {
    background: help-background;
    svg {
      fill: primary-text;
    }
  }
}
.row.help {
  margin-top: 0;
}

.reset {
  text-decoration: none;
  color: white;
  display: flex;
  justify-content: center;
}

.bounding-box {
  position: relative;
  .title {
    position: absolute;
    bottom: 0;
    font-size: 12px;
    left: 0;
    color: ternary-text;
  }
  .reset {
    font-size: 16px;
  }

  input[type='number'] {
    width: 100px;
    margin: 0;
    font-size: 12px;
    text-align: center;
    color: secondary-text;
  }
  input:invalid {
      box-shadow: none;
  }
  .max-x {
    justify-content: flex-end;
  }
}

@media (max-width: small-screen) {
  .settings {
    .title {
      font-size: 14px;
      text-align: left;
    }
  }
}

</style>
