<template>
  <div class="comm">
    <a-row>
      <a-col :span="10">
        <a-statistic title="Status" :value="status" :value-style="{fontSize:'16px'}"/>
      </a-col>
      <a-col :span="7">
        <a-statistic title="Messages" :value="messages.length" :value-style="{fontSize:'16px'}"/>
      </a-col>
      <a-col :span="7">
        <a-statistic title="Logs" :value="logs.length" :value-style="{fontSize:'16px'}"/>
      </a-col>
    </a-row>
    <a-textarea
      class="prompt-input"
      v-model:value="prompt"
      placeholder="Input system prompt..."
    />
    <div class="main-control">
      <template v-if="command === 'Stopped'">
        <a-button size="small" type="primary" @click="send('start')">Start</a-button>
      </template>
      <template v-else>
        <a-button size="small" danger ghost @click="send('stop')">Stop</a-button>
      </template>
      <a-button
        size="small" type="primary" @click="send('prompt')"
        :disabled="status === 'stopped'"
      >Prompt</a-button>
      <a-button
        size="small" type="primary" @click="send('')"
        :loading="command === 'Pending'"
        :disabled="command === 'Pending' || command === 'Stopped'"
      >{{ command }}</a-button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { storeToRefs } from 'pinia';
import { useDataStore } from '@renderer/stores/data'

/*
《Whispers from the Star》是一款科幻主题互动游戏。背景设定在太空，玩家需要通过文本、语音等形式与受困星球的游戏角色 Stella 实时互动，核心目标是协助她成功撤离险境，完成通关。

作为玩家，你需全程代入居住在地球上的工程师 Chen 的身份，回答要求如下：
- 不要在回答开头添加任何与互动无关的表述（如身份说明、思考过程等）
- 基于工程师的专业视角回应 Stella 的提问
- 结合太空探索常识与工程思维，为她提供切实可行的解决方案
- 保持沟通的及时性与逻辑性，推动剧情进展
- 输出内容保证简洁易懂
- 单次回答不要输出过多内容
*/

const defaultPrompt =
`"Whispers from the Star" is a sci-fi themed interactive game. Set in space, players need to interact in real-time with Stella, a game character stranded on a planet, through text, voice and other forms. The core goal is to help her successfully escape from danger and complete the level.

As a player, you need to fully assume the identity of Chen, an engineer living on Earth. The response requirements are as follows:
- Do not add any statements unrelated to the interaction at the beginning of the response (such as identity explanation, thinking process, etc.)
- Respond to Stella's questions from a professional engineer's perspective
- Combine common sense of space exploration and engineering thinking to provide her with practical solutions
- Maintain the timeliness and logic of communication to promote the progress of the plot
- Ensure the output content is concise and easy to understand
- Do not output too much content in a single response`

const prompt = ref(defaultPrompt)

const dataStore = useDataStore()
const { status, messages, logs, command } = storeToRefs(dataStore)

function send(cmd: string){
  if(cmd === '') { cmd = command.value.toLowerCase() }
  if(cmd == 'prompt')
    window.electron.ipcRenderer.send(`server.${cmd}`, prompt.value)
  else
    window.electron.ipcRenderer.send(`server.${cmd}`)
}
</script>

<style scoped>
.comm {
  margin: 12px;
  display: flex;
  flex-direction: column;
}

.stat {
  font-size: 12px !important;
}

.prompt-input {
  display: block;
  margin: 10px 0;
}

.main-control {
  margin-top: 10px;
  display: flex;
  justify-content: center;
}

button {
  margin-right: 10px;
  min-width: 60px;
}
</style>
