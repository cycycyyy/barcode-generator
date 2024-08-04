<script setup lang="ts">
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";
import {
  Select,
  SelectContent,
  SelectGroup,
  SelectItem,
  SelectLabel,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import { Checkbox } from "@/components/ui/checkbox";
import { useToast, Toaster } from "@/components/ui/toast";
import { ref, watch, h, onMounted } from "vue";
import JsBarcode from "jsbarcode";
import { TriangleAlert } from "lucide-vue-next";

const barcodeType = ref<string | null>("");
const isGenerateBarcodeForEachLine = ref(false);
const isEvaluateEscapeSequences = ref(true);
const { toast } = useToast();
const dataEntered = ref("");
const dataToBeGenerated = ref();
const whitespaceAppeared = ref(false);
const isGenerateBarcodeForEachLineVisible = ref(false);
const isEvaluateEscapeSequencesVisible = ref(false);

const stringToArray = (str: string) => {
  return str.split("\n");
};

const generateBarcode = () => {
  if (barcodeType.value) {
    if (dataEntered.value) {
      console.log("Data to be generated: ", dataToBeGenerated.value);
    } else {
      toast({
        title: "Oops!",
        description: "Please enter atleast one character in the text field.",
        variant: "destructive",
      });
    }
  } else {
    toast({
      title: "Oops!",
      description: "Please select a barcode type.",
      variant: "destructive",
    });
  }
};

const removeWhitespace = (str: string) => {
  // Define a regular expression to match whitespace excluding line breaks (\r and \n)
  const regex = /[ \t]+(?=[^\r\n]*[\r\n]|$)/g;

  // Replace matching whitespace with an empty string
  dataEntered.value = str.replace(regex, "");
};

const checkIfThereIsWhitespace = (str: string) => {
  return /[ \t]/.test(str);
};

watch(dataEntered, (newValue) => {
  if (newValue) {
    isGenerateBarcodeForEachLineVisible.value = true;
    isEvaluateEscapeSequencesVisible.value = true;
    if (checkIfThereIsWhitespace(newValue)) {
      toast({
        title: "Oops!",
        description: "There is a whitespace in the data entered.",
        variant: "destructive",
      });

      whitespaceAppeared.value = true;
    } else {
      whitespaceAppeared.value = false;
    }

    if (isGenerateBarcodeForEachLine.value) {
      dataToBeGenerated.value = stringToArray(dataEntered.value);
      previewBarcode(dataToBeGenerated.value[0]);
    } else {
      dataToBeGenerated.value = dataEntered.value;
      previewBarcode(dataToBeGenerated.value);
    }
  } else {
    isGenerateBarcodeForEachLineVisible.value = false;
    isEvaluateEscapeSequencesVisible.value = false;
    toast({
      title: "Oops!",
      description: "Please enter atleast one character.",
      variant: "destructive",
    });
  }
});

watch(isGenerateBarcodeForEachLine, (newValue) => {
  if (dataEntered.value) {
    if (newValue) {
      dataToBeGenerated.value = stringToArray(dataEntered.value);
      previewBarcode(dataToBeGenerated.value[0]);
    } else {
      dataToBeGenerated.value = dataEntered.value;
      previewBarcode(dataToBeGenerated.value);
    }
  } else {
    toast({
      title: "Oops!",
      description: "Please enter atleast one character.",
      variant: "destructive",
    });
  }
});

const previewBarcode = (str: string) => {
  const options = {
    format: "CODE128",
    width: 2,
    height: 100,
    displayValue: true,
    font: "arial",
  };

  // Preview the barcode
  JsBarcode("#barcode", str, options);
};
</script>

<template>
  <Toaster />
  <div class="h-screen w-full px-5 py-5">
    <div class="max-w-2xl mx-auto h-full">
      <span class="font-bold text-3xl text-center">Barcode Generator</span>

      <!-- Barcode Preview -->
      <div class="w-full flex justify-center h-[20vh] items-center flex-col">
        <svg id="barcode" v-show="dataEntered !== ''"></svg>
        <span
          class="text-destructive text-lg font-bold flex flex-col justify-center items-center"
          v-if="dataEntered === ''"
          ><TriangleAlert />No data entered</span
        >
      </div>

      <div class="flex flex-col gap-4 pt-5">
        <span class="font-bold">Choose a barcode type:</span>

        <Select v-model="barcodeType">
          <SelectTrigger class="w-[180px]">
            <SelectValue placeholder="Choose a barcode type" />
          </SelectTrigger>
          <SelectContent>
            <SelectGroup>
              <SelectLabel>Type</SelectLabel>
              <SelectItem value="code-128"> Code 128 </SelectItem>
            </SelectGroup>
          </SelectContent>
        </Select>

        <div class="flex justify-between items-center">
          <span class="font-bold py-3">Enter your data below:</span>
          <Button
            class="text-xs bg-destructive font-white"
            v-if="whitespaceAppeared"
            @click="removeWhitespace"
            >Trim Whitespace</Button
          >
        </div>

        <Textarea class="h-[200px]" v-model="dataEntered"></Textarea>

        <div class="space-y-2 pb-5">
          <div class="flex items-center space-x-2">
            <Checkbox
              id="generate-a-barcode-for-each-line"
              v-model:checked="isGenerateBarcodeForEachLine"
              :disabled="!isGenerateBarcodeForEachLineVisible"
            />
            <label
              for="generate-a-barcode-for-each-line"
              class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
            >
              Generate a barcode for each line
            </label>
          </div>
          <div class="flex items-center space-x-2">
            <Checkbox
              id="evaluate-escape-sequences"
              v-model:checked="isEvaluateEscapeSequences"
              :disabled="!isEvaluateEscapeSequencesVisible"
            />
            <label
              for="evaluate-escape-sequences"
              class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
            >
              Evaluate escape sequences
            </label>
            <span class="text-sm text-gray-500"
              >Ex. \F for FNC1, \t for TAB, \n for ENTER</span
            >
          </div>
        </div>

        <Button class="h-12 text-lg" @click="generateBarcode">Generate</Button>
      </div>
    </div>
  </div>
</template>
