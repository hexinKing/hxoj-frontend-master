<template>
  <div class="home">
    <CodeEditor :handle-change="onCodeChange" :value="codeValue" />
    <MdEditor :handle-change="onMdChange" :value="mdValue" />
  </div>
</template>

<script lang="ts" setup>
import { ref } from "vue";
import MdEditor from "@/components/MdEditor.vue";
import CodeEditor from "@/components/CodeEditor.vue"; // @ is an alias to /src

const mdValue = ref();
const codeValue = ref(
  "public class Main {\n" +
    "    public static void main(String[] args) {\n" +
    '        System.out.println("Hello, World!");\n' +
    "    }\n" +
    "}\n"
);

const onMdChange = (v: string) => {
  mdValue.value = v;
  console.log("mdValue:", v);
};

const onCodeChange = (v: string) => {
  codeValue.value = v;
  console.log("codeValue:", v);
};
</script>
