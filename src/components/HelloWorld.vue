<template>
  <input type="file" accept=".xlsx" ref="uploadInputRef" @change="uploadFileChange" style="display: none">
  <a :href="xmlfilelink" :download="xmlfilename" ref="downloadXmlRef" style="display: none;"></a>
  <article class="panel">
    <p class="panel-heading has-text-centered is-size-6 has-text-weight-normal">
      Отчет о клиентах-резидентах иностранных государств 5.04
    </p>

    <div class="panel-block">
      <div class="control">
        <div class="field has-addons">
          <div class="control">
            <div class="button is-static">
              1.
            </div>
          </div>
          <div class="control is-expanded">
            
            <button @click="uploadClick" :class="[{'is-info': !file, 'is-outlined': file, 'is-loading': loading},'button is-fullwidth']" is-focused="false">Загрузить XLSX таблицу...</button>
          </div>
        </div>
      </div>
    </div>

    <div class="panel-block">
      <div class="control">
        <div class="field has-addons">
          <div class="control">
            <div :class="[{'has-text-grey-lighter': !file},'button is-static']">
              2.
            </div>
          </div>
          <div class="control is-expanded">
            <button @click="getReportClick" :class="[{'is-info': file},'button  is-fullwidth']" :disabled="!file">Сформировать и скачать отчет в XML формате</button>
          </div>
        </div>
      </div>
    </div>
  </article>
    <transition name="slide-fade">
    <div v-if="lastoperation" class="message is-white is-small">
      <div class="message-body p-1 has-text-grey-light" v-html="lastoperation">
      </div>
    </div>
    </transition>
  </template>

<script setup>
  import { nextTick, ref } from 'vue'
  
  const lastoperation = ref(null) 

  const xmlfilelink = ref(null) 
  const xmlfilename = ref(null) 

  // серверное имя файла (полученное после загрузки файла)
  const file = ref(null) 

  // индикатор загрузки файла
  const loading = ref(false)

  // кнопка "Выбрать XLSX таблицу..."
  const uploadInputRef = ref(null)
  const uploadClick = async event => { 
    event.target.blur()
    await nextTick()
    uploadInputRef.value.value = ''
    lastoperation.value = ''
    uploadInputRef.value.click() 
  }

  // скрытое поле  input type=«file» - используется для загрузки файла
  // вызывается после загрузки файла
  const uploadFileChange = event => {
    const files = event.target.files
    if (files.length > 0) {
      file.value = null
      loading.value = true
      const formData = new FormData()
      formData.append('file', files[0])   
      fetch('/api/upload', { 
        method: 'POST',
        body: formData
      }).then(
        response => {
          if (response.ok)
            return response.json()
          throw new Error('Ошибка загрузки отчета');
        }
      )
      .then(success => { 
          file.value = success.filename
          lastoperation.value = 'Отчет загружен'
          loading.value = false
        }
      ).catch(async error => {
          loading.value = false
          lastoperation.value = error.message
        }
      )
    }
  }

  const downloadXmlRef = ref(null)
  // кнопка "Скачать отчет в XML формате"
  const getReportClick = async event => {
    event.target.blur()
    await nextTick()
    lastoperation.value = ''
    fetch('/api/getreport/' + file.value, { 
      method: 'GET'
    }).then(
      response => {  
        if (response.ok)
          return response.blob()       
        throw new Error('Ошибка формирования отчета');
      }
    ).then(async blob => {
      const url = window.URL.createObjectURL(blob);
      xmlfilelink.value = url
      xmlfilename.value = file.value
      await nextTick()
      downloadXmlRef.value.click()
      window.URL.revokeObjectURL(url);
      lastoperation.value = 'Отчет сформирован в файл <a href="/api/getreport/' + file.value + '">' + file.value + '</a>'
      file.value = ''  
    }).catch(async error => {
      lastoperation.value = error.message
    })
  }

</script>

<style scoped>
  .slide-fade-enter-active {
    transition: all 0.3s ease-out;
  }

  .slide-fade-leave-active {
    transition: all 0.8s cubic-bezier(1, 0.5, 0.8, 1);
  }

  .slide-fade-enter-from,
  .slide-fade-leave-to {
    transform: translateY(-20px);
    opacity: 0;
  }
</style>