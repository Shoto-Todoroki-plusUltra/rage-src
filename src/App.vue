<template>
  <div id="app" @click="spawnEmoji">
    <GridBackground ref="gridBackground" @click.native="spawnEmoji" />
    
    <main class="content-container">
      <transition name="fade-scale" mode="out-in">
        <div>
        <div v-if="!isProcessing" key="upload-interface" class="main-content">
          <h1 class="cyber-title glitch">
            <div class="playwrite-it-moderna-regular">ρage</div>
            <div class="playwrite-it-moderna-regular">deplagiarize like a slayy</div>
          </h1>
          
          <div class="upload-container"
               @dragover.prevent="isDragging = true"
               @dragleave.prevent="isDragging = false"
               @drop.prevent="handleFileDrop">
            <div class="upload-box ribbon-cursor" :class="{ 'drag-over': isDragging }">
              <div class="upload-content">
                <upload-icon class="upload-icon" />
                <p class="upload-text neon-text">drop em docs already QWEEENN💅🏻😩</p>
              </div>
              <input type="file" 
                     ref="fileInput" 
                     @change="handleFileUpload" 
                     class="hidden-input"
                     accept=".docx,.txt, .doc" />
            </div>
          </div>
          <div id="status" style="display: hidden;"></div>
        </div>
      </div>
      </transition>
    </main>

  </div>
</template>

<script>
import GridBackground from './components/GridBackground.vue'
import mammoth from 'mammoth';
import { saveAs } from 'file-saver';
import JSZip from 'jszip';

export default {
  name: 'App',
  components: {
    GridBackground
  },
  data() {
    return {
      isDragging: false,
      processing: false,
      file: null,
      fileContent: null,
      substitutionMap: {
        'a': 'а',
        'c': 'с',
        'd': 'ԁ',
        'e': 'е'
      }
    }
},
  mounted() {
    document.body.classList.add('custom-cursor');
  },
  methods: {
    handleFileDrop(e) {
      const files = e.dataTransfer.files;
      if (files.length) {
        this.processFile(files[0]);
      }
      this.isDragging = false;
    },
    handleFileUpload(e) {
      const files = e.target.files;
      if (files.length) {
        this.processFile(files[0]);
      }
    },

    processFile(file) {
      const validExtensions = ['.doc', '.docx', '.txt'];
      const fileName = file.name.toLowerCase();
      const fileExtension = '.' + fileName.split('.').pop();

      if (!validExtensions.includes(fileExtension)) {
        alert('Please upload a .doc, .docx, or .txt file');
        return;
      }

      this.file = file;
      this.isProcessing = true;

      if (fileExtension === '.txt') {
        this.processTxtFile(file);
      } else if (fileExtension === '.docx') {
        this.processDocxFile(file);
      } else if (fileExtension === '.doc') {
        alert('Legacy .doc files require conversion to .docx for best results');
        this.isProcessing = false;
      }
    },

    processTxtFile(file) {
      const reader = new FileReader();

      reader.onload = (e) => {
        const content = e.target.result;
        const modifiedContent = this.substituteText(content);
        this.downloadTxtFile(modifiedContent, file.name);
        this.isProcessing = false;
      };

      reader.onerror = () => {
        alert('Error reading file');
        this.isProcessing = false;
      };

      reader.readAsText(file);
    },

    processDocxFile(file) {
      const reader = new FileReader();

      reader.onload = async (e) => {
        try {
          const arrayBuffer = e.target.result;
          const result = await mammoth.extractRawText({
            arrayBuffer
          });
          const extractedText = result.value;

          const modifiedText = this.substituteText(extractedText);

          await this.modifyDocxFile(arrayBuffer, file.name);

          this.isProcessing = false;
        } catch (error) {
          console.error('Error processing DOCX file:', error);
          alert('Error processing the file');
          this.isProcessing = false;
        }
      };

      reader.onerror = () => {
        alert('Error reading file');
        this.isProcessing = false;
      };

      reader.readAsArrayBuffer(file);
    },

    async modifyDocxFile(arrayBuffer, filename) {
      try {
        const zip = await JSZip.loadAsync(arrayBuffer);

        const safeReplaceInXml = (xmlContent) => {
          const parser = new DOMParser();
          const xmlDoc = parser.parseFromString(xmlContent, 'text/xml');

          const textNodes = xmlDoc.getElementsByTagName('w:t');

          for (let node of textNodes) {
            let text = node.textContent;

            Object.keys(this.substitutionMap).forEach(originalChar => {
              const replacement = this.substitutionMap[originalChar];

              text = text.split(originalChar).join(replacement);
            });

            node.textContent = text;
          }

          const serializer = new XMLSerializer();
          return serializer.serializeToString(xmlDoc);
        };

        const documentXml = await zip.file('word/document.xml').async('string');
        const modifiedDocumentXml = safeReplaceInXml(documentXml);
        zip.file('word/document.xml', modifiedDocumentXml);

        const modifiedDocxBlob = await zip.generateAsync({
          type: 'blob',
          mimeType: 'application/vnd.openxmlformats-officedocument.wordprocessingml.document'
        });

        const filenameParts = filename.split('.');
        const extension = filenameParts.pop();
        const nameWithoutExtension = filenameParts.join('.');
        const newFilename = `${nameWithoutExtension}_clean.${extension}`;

        saveAs(modifiedDocxBlob, newFilename);

      } catch (error) {
        console.error('DOCX modification error:', error);
        alert('Error processing the document');
      }
    },

    escapeRegExp(string) {
      return string.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
    },

    substituteText(text) {
      let modifiedText = text;

      Object.keys(this.substitutionMap).forEach(originalChar => {
        const replacement = this.substitutionMap[originalChar];

        const regex = new RegExp(this.escapeRegExp(originalChar), 'g');
        modifiedText = modifiedText.replace(regex, replacement);
      });

      return modifiedText;
    },

    downloadTxtFile(content, filename) {
      const filenameParts = filename.split('.');
      const extension = filenameParts.pop();
      const nameWithoutExtension = filenameParts.join('.');
      const newFilename = `${nameWithoutExtension}_clean.${extension}`;

      const blob = new Blob([content], {
        type: 'text/plain'
      });
      saveAs(blob, newFilename);
    },

    spawnEmoji(event) {
      if (event.target.closest('.upload-box') || event.target.closest('.hidden-input')) {
        return;
      }

      if (this.$refs.gridBackground) {
        this.$refs.gridBackground.addParticleAtPosition(event.clientX, event.clientY);
      }
    }
  }
}
</script>

<style scoped>
.grid-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  overflow: hidden;
}

.grid-line {
  position: absolute;
  background: rgba(46, 230, 214, 0.05);
}

.horizontal {
  width: 100%;
  height: 1px;
  animation: moveHorizontal 40s linear infinite;
}

.vertical {
  height: 100%;
  width: 1px;
  animation: moveVertical 40s linear infinite;
}

.grid-particles {
  position: absolute;
  width: 100%;
  height: 100%;
}

.grid-particle {
  position: absolute;
  width: 8px;
  height: 8px;
  background: var(--cyber-teal);
  border-radius: 2px;
  transform: rotate(45deg);
  animation: particleGlow 4s ease-in-out infinite;
}

@keyframes moveHorizontal {
  0% { transform: translateY(-50%); }
  100% { transform: translateY(50%); }
}

@keyframes moveVertical {
  0% { transform: translateX(-50%); }
  100% { transform: translateX(50%); }
}

@keyframes particleGlow {
  0%, 100% { 
    transform: rotate(45deg) scale(0.8);
    opacity: 0.2;
  }
  50% { 
    transform: rotate(45deg) scale(1.2);
    opacity: 0.4;
  }
}
</style>
