<template>
  <main v-if="!loading" id="geometry-page" class="page  bg-black text-white min-h-screen">
    <div id="mobile-page-title" class="mb-4">
      <h2>_geometriya</h2>
    </div>

    <div id="page-menu" class="w-full flex">
      <!-- SECTIONS (use key and value) -->
      <div id="sections">
        <div id="section-icon" v-for="(section, sectionKey) in config.about.sections" :key="sectionKey"
          :class="{ active: isSectionActive(sectionKey) }">
          <img :id="'section-icon-' + sectionKey" :src="section.icon" :alt="section.title + '-section'"
            @click="focusCurrentSection(sectionKey)" />
        </div>
      </div>

      <!-- focused section content (desktop) -->
      <div id="section-content" class="hidden lg:block w-full h-full border-right">
        <!-- title -->
        <div id="section-content-title" class="hidden lg:flex items-center min-w-full">
          <img id="section-arrow-menu" src="/icons/arrow.svg" alt="" class="section-arrow mx-3 open" />
          <p v-html="config.about.sections[currentSection]?.title" class="font-fira_regular text-white text-sm"></p>
        </div>

        <!-- folders (use key and folder object) -->
        <div>
          <div v-for="(folder, folderKey, index) in config.about.sections[currentSection]?.info" :key="folderKey"
            class="grid grid-cols-2 items-center my-2 font-fira_regular text-menu-text"
            @click="focusCurrentFolder(folderKey)">
            <div class="flex col-span-2 hover:text-white hover:cursor-pointer">
              <img id="diple" src="/icons/diple.svg" alt="" :class="{ open: isOpen(folderKey) }" />
              <img :src="'/icons/folder' + (index + 1) + '.svg'" alt="" class="mr-3" />
              <!-- display human title, but keep id safe (use folderKey) -->
              <p :id="folderKey" v-html="folder.title" :class="{ active: isActive(folderKey) }"></p>
            </div>

            <div v-if="folder.files !== undefined" class="col-span-2">
              <div v-for="(fileContent, fileName) in folder.files" :key="fileName"
                class="hover:text-white hover:cursor-pointer flex my-2" @click.stop="openFile(folderKey, fileName)">
                <img src="/icons/markdown.svg" alt="" class="ml-8 mr-3" />
                <p>{{ fileName }}</p>
              </div>
            </div>
          </div>
        </div>

        <!-- contact -->
        <div id="section-content-title-contact" class="flex items-center min-w-full border-top">
          <img id="section-arrow-menu" src="/icons/arrow.svg" alt="" class="section-arrow mx-3 open" />
          <p v-html="config.contacts.direct.title" class="font-fira_regular text-white text-sm"></p>
        </div>

        <div id="contact-sources" class="hidden lg:flex lg:flex-col my-2">
          <div v-for="(source, key) in config.contacts.direct.sources" :key="key" class="flex items-center mb-2">
            <img :src="'/icons/' + key + '.svg'" alt="" class="mx-4" />
            <a v-html="source" href="/" class="font-fira_retina text-menu-text hover:text-white"></a>
          </div>
        </div>
      </div>

      <!-- mobile -->
      <div id="section-content" class="lg:hidden w-full font-fira_regular">
        <div v-for="(section, sectionKey) in config.about.sections" :key="sectionKey">
          <!-- section title (mobile) -->
          <div :key="sectionKey" :src="section.icon" id="section-content-title" class="flex lg:hidden mb-1"
            @click="focusCurrentSection(sectionKey)">
            <img src="/icons/arrow.svg" :id="'section-arrow-' + sectionKey" alt="" class="section-arrow" />
            <p v-html="section.title" class="text-white text-sm"></p>
          </div>

          <!-- folders -->
          <div :id="'folders-' + sectionKey" class="hidden">
            <div v-for="(folder, folderKey, index) in config.about.sections[sectionKey]?.info" :key="folderKey"
              class="grid grid-cols-2 items-center my-2 font-fira_regular text-menu-text hover:text-white hover:cursor-pointer"
              @click="focusCurrentFolder(folderKey)">
              <div class="flex col-span-2">
                <img id="diple" src="/icons/diple.svg" />
                <img :src="'icons/folder' + (index + 1) + '.svg'" alt="" class="mr-3" />
                <p :id="folderKey" v-html="folder.title" :class="{ active: isActive(folderKey) }"></p>
              </div>

              <div v-if="folder.files !== undefined" class="col-span-2">
                <div v-for="(fileContent, fileName) in folder.files" :key="fileName"
                  class="hover:text-white hover:cursor-pointer flex my-2" @click.stop="openFile(folderKey, fileName)">
                  <img src="/icons/markdown.svg" alt="" class="ml-8 mr-3" />
                  <p>{{ fileName }}</p>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- section content title -->
        <div id="section-content-title" class="flex items-center min-w-full" @click="showContacts">
          <img src="/icons/arrow.svg" alt="" id="section-arrow" class="section-arrow" />
          <p v-html="config.contacts.direct.title" class="font-fira_regular text-white text-sm"></p>
        </div>

        <!-- section content folders -->
        <div id="contacts" class="hidden">
          <div v-for="(source, key) in config.contacts.direct.sources" :key="key" class="flex items-center my-2">
            <img :src="'/icons/' + key + '.svg'" alt="" />
            <a v-html="source" href="/" class="font-fira_retina text-menu-text hover:text-white ml-4"></a>
          </div>
        </div>
      </div>
    </div>

    <!-- content -->
    <div class="flex flex-col lg:grid lg:grid-cols-1 h-full w-full">
      <div id="left" class="w-full flex flex-col border-right">
        <!-- windows tab desktop -->
        <div class="tab-height w-full hidden lg:flex border-bot items-center">
          <div class="flex items-center border-right h-full">
            <p v-html="config.about.sections[currentSection]?.title"
              class="font-fira_regular text-menu-text text-sm px-3"></p>
            <img src="/icons/close.svg" alt="" class="mx-3" />
          </div>
        </div>

        <!-- windows tab mobile -->
        <div id="tab-mobile" class="flex lg:hidden font-fira_retina">
          <span class="text-white">// </span>
          <h3 v-html="config.about.sections[currentSection]?.title" class="text-white px-2"></h3>
          <span class="text-menu-text"> / </span>
          <h3 v-html="config.about.sections[currentSection]?.info[folder].title" class="text-menu-text pl-2"></h3>
        </div>

        <!-- text / interactive content -->
        <div id="commented-text" class="flex h-full w-full lg:border-right overflow-hidden">
          <div class="w-full h-full ml-5 mr-10 lg:my-5 overflow-scroll">
            <!-- If folder has a component -> render it, else show description with CommentedText -->
            <component v-if="currentComponent" :is="currentComponent" />
            <CommentedText v-else :text="currentDescription" />
          </div>

          <!-- scroll bar -->
          <div id="scroll-bar" class="h-full border-left hidden lg:flex justify-center py-1">
            <div id="scroll"></div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script>
import DevConfig from '~/developer.json';
import TriangleAreaBySine from '~/components/TriangleAreaBySine.vue';
import LawOfSines from '~/components/LawOfSines.vue';

export default {
  components: { TriangleAreaBySine, LawOfSines },
  data() {
    return {
      config: DevConfig,
      currentSection: 'geometry',
      folder: 'triangle-area',
      loading: true,
      currentComponent: null,
      currentDescription: ''
    };
  },
  computed: {
    isActive() {
      return (folderKey) => this.folder === folderKey;
    },
    isSectionActive() {
      return (sectionKey) => this.currentSection === sectionKey;
    },
    isOpen() {
      return (folderKey) => this.folder === folderKey;
    }
  },
  methods: {
    focusCurrentSection(sectionKey) {
      // sectionKey is the key, e.g. 'geometry'
      this.currentSection = sectionKey;
      const section = this.config.about.sections[sectionKey];
      const firstKey = Object.keys(section.info)[0];
      this.folder = firstKey;

      // toggle mobile folders/arrow if present
      const el = document.getElementById('folders-' + sectionKey);
      if (el) el.classList.toggle('hidden');
      const arrow = document.getElementById('section-arrow-' + sectionKey);
      if (arrow) arrow.classList.toggle('rotate-90');

      this.loadFolderContent();
    },
    focusCurrentFolder(folderKey) {
      // folderKey is the key, e.g. 'law-of-sines'
      this.folder = folderKey;

      // find section that contains this folder (safeguard) and set currentSection
      for (const sk of Object.keys(this.config.about.sections)) {
        const sec = this.config.about.sections[sk];
        if (sec && sec.info && sec.info[folderKey]) {
          this.currentSection = sk;
          break;
        }
      }

      this.loadFolderContent();
    },
    openFile(folderKey, fileName) {
      const f = this.config.about.sections[this.currentSection].info[folderKey].files[fileName];
      if (typeof f === 'string') {
        this.currentComponent = null;
        this.currentDescription = f;
      }
    },
    toggleFiles() {
      const el = document.getElementById('file-' + this.folder);
      if (el) el.classList.toggle('hidden');
    },
    showContacts() {
      const contactsEl = document.getElementById('contacts');
      if (contactsEl) contactsEl.classList.toggle('hidden');
      const arrow = document.getElementById('section-arrow');
      if (arrow) arrow.classList.toggle('rotate-90');
    },
    loadFolderContent() {
      // Map folder keys to actual imported components
      if (this.folder === 'triangle-area') {
        this.currentComponent = TriangleAreaBySine;
        this.currentDescription = '';
      } else if (this.folder === 'law-of-sines') {
        this.currentComponent = LawOfSines;
        this.currentDescription = '';
      } else {
        this.currentComponent = null;
        this.currentDescription =
          (this.config.about.sections[this.currentSection]?.info[this.folder]?.description) || '';
      }
    }
  },
  mounted() {
    // Ensure geometry fallback exists
    if (!this.config.about) this.config.about = { sections: {} };
    if (!this.config.about.sections) this.config.about.sections = {};
    if (!this.config.about.sections.geometry) {
      this.config.about.sections.geometry = {
        title: 'Geometriya',
        icon: '/icons/geometry.svg',
        info: {
          'triangle-area': {
            title: 'Uchburchak yuzini hisoblash',
            description: "A = 1/2 · a · b · sin(C) formulasi bilan interaktiv hisoblagich uchun komponent mavjud.",
            files: { 'Kirish.md': "Ushbu bo'limda A = 1/2 a b sin C formulasi tushuntiriladi." }
          },
          'law-of-sines': {
            title: 'Sinuslar teoremasi',
            description: 'a/sinA = b/sinB = c/sinC — interaktiv yechim komponenti mavjud.',
            files: { 'Kirish.md': 'Sinuslar teoremasining qisqacha izohi.' }
          }
        }
      };
    }

    // load initial folder component/description
    this.loadFolderContent();
    this.loading = false;
  }
};
</script>

<style scoped>
/* ... (sizning mavjud style qismi shu yerda) ... */
#sections {
  width: 5rem;
  height: 100%;
  display: none;
  border-right: 1px solid #1E2D3D;
}

@media (min-width: 1024px) {
  #sections {
    display: block;
  }
}

#section-icon {
  @apply my-6 hover:cursor-pointer flex justify-center;
  opacity: 0.4;
}

#section-icon.active {
  opacity: 1;
}

#section-icon:hover {
  opacity: 1;
}

.tab-height {
  min-height: 35px;
  max-height: 35px;
}

#tab-mobile {
  padding: 25px 20px 0px 25px;
  align-items: flex-end;
}

#scroll-bar {
  width: 20px;
}

#scroll {
  width: 14px;
  height: 7px;
  background-color: #607B96;
}

#diple {
  @apply mx-3 w-2 max-w-fit;
}

.open {
  transform: rotate(90deg);
}

.active {
  color: white;
}

#right,
#left {
  height: 100%;
  overflow: hidden;
}

#gists-content {
  height: 100%;
  overflow: hidden;
}

.section-arrow {
  transition: 0.1s;
}

#section-content #contacts {
  padding: 0px 25px;
}
</style>
