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
import { Input } from "@/components/ui/input";
import { ref, watch } from "vue";
import JsBarcode from "jsbarcode";
import JSZip from "jszip";
import { saveAs } from "file-saver";
import { TriangleAlert, FolderSync } from "lucide-vue-next";

const barcodeType = ref<string | undefined>("CODE128");
const isGenerateBarcodeForEachLine = ref(false);
// const isEvaluateEscapeSequences = ref(true);
const { toast } = useToast();
const dataEntered = ref("");
const dataToBeGenerated = ref();
const whitespaceAppeared = ref(false);
const isGenerateBarcodeForEachLineVisible = ref(false);
const isEvaluateEscapeSequencesVisible = ref(false);
const fileTypeChoice = ref("png");
const dpi = ref(96);

const stringToArray = (str: string) => {
  return str.split("\n");
};

const generateBarcode = () => {
  if (barcodeType.value) {
    if (dataEntered.value) {
      console.log(fileTypeChoice.value, dpi.value);
      if (fileTypeChoice.value === "png" && dpi.value >= 96) {
        exportMultipleBarcodesAsZip();
      } else if (fileTypeChoice.value === "svg") {
        exportMultipleBarcodesAsZip();
      } else {
        toast({
          title: "Oops!",
          description: "Please enter your desired DPI. Minimum of 96 DPI.",
          variant: "destructive",
        });
      }
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

/* const removeWhitespace = (str: string) => {
  // Define a regular expression to match whitespace excluding line breaks (\r and \n)
  const regex = /[ \t]+(?=[^\r\n]*[\r\n]|$)/g;

  // Replace matching whitespace with an empty string
  dataEntered.value = str.replace(regex, "");
}; */

const checkIfThereIsWhitespace = (str: string) => {
  return /[ \t]/.test(str);
};

const refreshPreview = () => {
  isGenerateBarcodeForEachLineVisible.value = true;
  isEvaluateEscapeSequencesVisible.value = true;
  if (checkIfThereIsWhitespace(dataEntered.value)) {
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

  if (!dataEntered.value) {
    isGenerateBarcodeForEachLineVisible.value = false;
    isEvaluateEscapeSequencesVisible.value = false;
    toast({
      title: "Oops!",
      description: "Please enter atleast one character.",
      variant: "destructive",
    });
  }
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
  try {
    const options = {
      format: String(barcodeType.value),
      width: 2,
      height: 100,
      displayValue: true,
      font: "arial",
    };

    // Preview the barcode
    JsBarcode("#barcode", str, options);
  } catch (error) {
    toast({
      title: `${error}`,
      variant: "destructive",
    });
  }
};
const exportMultipleBarcodesAsZip = async () => {
  const zip = new JSZip();
  const barcodeDataArray = ref<string[]>([]);

  if (isGenerateBarcodeForEachLine.value) {
    barcodeDataArray.value = stringToArray(dataEntered.value);
  } else {
    barcodeDataArray.value.push(dataEntered.value);
  }

  for (let i = 0; i < barcodeDataArray.value.length; i++) {
    console.log("Generating barcode data for: ", barcodeDataArray.value[i]);
    const svgElement = document.createElementNS(
      "http://www.w3.org/2000/svg",
      "svg"
    ); // Use createElementNS for SVG elements
    const options = {
      format: String(barcodeType.value),
      width: 2,
      height: 100,
      displayValue: true,
      font: "arial",
    };

    JsBarcode(svgElement, barcodeDataArray.value[i], options);

    const serializer = new XMLSerializer();
    let svgString = serializer.serializeToString(svgElement);

    // Clean up the SVG string to remove duplicate xmlns attributes
    svgString = cleanSVG(svgString);

    if (fileTypeChoice.value === "png") {
      // Convert SVG string to PNG blob
      const pngBlob = await convertSvgToPngBlob(svgString);
      zip.file(`${barcodeDataArray.value[i]}.png`, pngBlob);
    } else {
      zip.file(`${barcodeDataArray.value[i]}.svg`, svgString);
    }
  }

  const blob = await zip.generateAsync({ type: "blob" });
  saveAs(blob, "barcodes.zip");
};

// Function to convert SVG string to PNG blob
const convertSvgToPngBlob = (svgString: string): Promise<Blob> => {
  return new Promise((resolve, reject) => {
    const svgImage = new Image();
    const canvas = document.createElement("canvas");
    const context = canvas.getContext("2d");

    svgImage.onload = () => {
      // Calculate the canvas size based on DPI
      const scaleFactor = dpi.value / 96; // 96 DPI is the default for most browsers
      const width = svgImage.width * scaleFactor;
      const height = svgImage.height * scaleFactor;

      canvas.width = width;
      canvas.height = height;

      if (context) {
        // Adjust the scaling for DPI
        context.scale(scaleFactor, scaleFactor);
        context.drawImage(svgImage, 0, 0);

        canvas.toBlob((blob) => {
          if (blob) {
            resolve(blob);
          } else {
            reject(new Error("Conversion to PNG failed."));
          }
        });
      } else {
        reject(new Error("Failed to get canvas context."));
      }
    };

    svgImage.onerror = (error) => reject(error);
    svgImage.src =
      "data:image/svg+xml;charset=utf-8," + encodeURIComponent(svgString);
  });
};

// Function to clean up the SVG string
const cleanSVG = (svgString: string) => {
  // Remove duplicate xmlns attributes
  const cleanedSVG = svgString.replace(
    /xmlns="http:\/\/www.w3.org\/2000\/svg"\s*/g,
    ""
  );

  // Add a single xmlns attribute at the beginning
  return cleanedSVG.replace("<svg", '<svg xmlns="http://www.w3.org/2000/svg"');
};
</script>

<template>
  <Toaster />
  <div class="w-full px-5 py-5">
    <div class="max-w-2xl mx-auto h-full">
      <span class="font-bold text-3xl text-center">Barcode Generator v1.0</span>

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
            <SelectValue placeholder="Barcode Type" />
          </SelectTrigger>
          <SelectContent>
            <SelectGroup>
              <SelectLabel>Type</SelectLabel>
              <SelectItem value="CODE128"> Code 128 </SelectItem>
              <SelectItem value="EAN13"> EAN-13 </SelectItem>
              <SelectItem value="UPC"> UPC </SelectItem>
              <SelectItem value="EAN8"> EAN-8 </SelectItem>
              <SelectItem value="EAN5"> EAN-5 </SelectItem>
              <SelectItem value="EAN2"> EAN-2 </SelectItem>
              <SelectItem value="CODE39"> CODE39 </SelectItem>
              <SelectItem value="ITF14"> ITF14 </SelectItem>
              <SelectItem value="MSI"> MSI </SelectItem>
              <SelectItem value="MSI10"> MSI-10 </SelectItem>
              <SelectItem value="MSI11"> MSI-11 </SelectItem>
              <SelectItem value="MSI1010"> MSI-1010 </SelectItem>
              <SelectItem value="MSI1110"> MSI-1110 </SelectItem>
              <SelectItem value="pharmacode"> Pharmacode </SelectItem>
              <SelectItem value="codabar"> Codabar </SelectItem>
            </SelectGroup>
          </SelectContent>
        </Select>

        <div class="flex justify-between items-center">
          <span class="font-bold">Enter your data below:</span>
          <Button
            class="flex items-center gap-2 justify-center text-xs font-white"
            @click="refreshPreview"
            ><FolderSync class="size-4" /> <span>Refresh</span></Button
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
          <!-- <div class="flex items-center space-x-2">
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
          </div> -->
        </div>
        <span class="font-bold">Choose a file type:</span>

        <Select v-model="fileTypeChoice">
          <SelectTrigger class="w-[180px]">
            <SelectValue placeholder="File Type" />
          </SelectTrigger>
          <SelectContent>
            <SelectGroup>
              <SelectLabel>Type</SelectLabel>
              <SelectItem value="png"> PNG </SelectItem>
              <SelectItem value="svg"> SVG </SelectItem>
            </SelectGroup>
          </SelectContent>
        </Select>

        <span class="font-bold" v-if="fileTypeChoice === 'png'">Set DPI:</span>
        <Input v-model="dpi" v-if="fileTypeChoice === 'png'" type="number" />

        <Button class="h-12 text-lg" @click="generateBarcode">Generate</Button>
      </div>
    </div>
  </div>
</template>
