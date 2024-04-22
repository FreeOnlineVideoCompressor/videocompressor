<template>
  <div class="mt-8 flex justify-center px-4 py-6 sm:p-8 rounded-lg shadow-lg bg-202020 w-full max-w-6xl mx-auto">
    <div class="w-full max-w-md">
      <div v-if="!store.state.ffFileOk">
        <div class="pb-6">
          <div class="pb-2 font-bold">Upload a file</div>
          <input
            class="form-control block w-full text-sm font-light bg-161616/50 rounded cursor-pointer file:py-2 file:px-4 file:mr-4 file:rounded-none file:bg-161616 file:border-none file:text-sm file:font-normal file:text-gray-200"
            type="file" accept="video/mp4" v-on:change="onFileChanged" />
          <div class="pt-2 pb-2 text-sm text-gray-300">
            <span class="text-xs px-3 py-1 bg-161616/50 text-gray-300 rounded select-none mr-1">Original</span>
            {{ store.state.ffFilesize ? sizeBytes(Number(store.state.ffFilesize)) : "-" }}
            <span class="text-xs px-3 py-1 bg-161616/50 text-gray-300 rounded select-none ml-2 mr-1">Minimum</span>
            {{ minimumSize ? sizeBytes(Number(minimumSize)) : "-" }}
          </div>
        </div>
        <div class="pb-6">
          <div class="pb-2 font-bold">Choose your desired file size for video compression</div>
          <input class="bg-161616 rounded px-3 py-2 w-24 text-sm" type="number" v-model="targetSize" />
          MB
        </div>
        <div class="pb-6">
          <div class="pb-2 font-bold">Choose video resolution</div>
          <div class="flex flex-wrap gap-x-2 gap-y-1">
            <div v-for="res in resScale" :key="res" :id="'res-' + String(res)">
              <button class="px-3 py-2 text-xs text-gray-200 transition duration-150 rounded"
                :class="String(targetScale) == String(res) ? 'bg-indigo-800/80 hover:bg-indigo-800/70' : 'bg-indigo-800/30 hover:bg-indigo-800/50'"
                @click="targetScale = res">{{ res }}x</button>
            </div>
          </div>
          <div class="pt-2 pb-2 text-sm text-gray-300" v-if="targetScaleAuto">We recommend decreasing video resolution
            for better results :')</div>
        </div>
        <div class="pb-2 text-right">
          <button
            class="bg-161616/50 transition-colors duration-150 px-4 py-2 rounded text-gray-200 cursor-default select-none"
            v-if="store.state.ffFileOkPre">
            <span class="inline-block animate-pulse">⏳&ensp;Compress</span>
          </button>
          <button
            class="bg-indigo-800 hover:bg-indigo-700 transition-colors duration-150 px-4 py-2 rounded text-gray-200 select-none"
            v-on:click="onSubmit" v-else>
            Compress
          </button>
        </div>
      </div>
      <div v-else>
        <div class="px-3 py-2 block text-sm bg-161616/25 rounded mt-2 cursor-default">
          {{ store.state.ffFilename }}
        </div>
        <div class="px-3 py-2 block text-sm bg-161616/25 rounded mt-2 cursor-default">
          {{ sizeBytes(Number(store.state.ffFilesize)) }} ->
          {{ sizeBytes(Number(targetSize * 1024 * 1024)) }}
        </div>
        <div class="px-3 py-2 block text-sm bg-161616/25 rounded mt-2 cursor-default">
          {{ targetScale }}x resolution
        </div>
        <div class="pt-2 pb-2 text-right" title="Cancel">
          <button class="bg-transparent px-4 py-2 rounded text-gray-200" v-if="store.state.ffProgress < 100"
            v-on:click="reload()">
            ❌
          </button>
          <button class="bg-indigo-800 shadow px-4 py-2 rounded text-gray-200" v-else v-on:click="reload()">
            Compress new video
          </button>
        </div>
        <div class="pt-2 pb-2" v-if="store.state.consoleErr">
          <span class="absolute transition-all duration-150">⚠</span>
          <span class="ml-8">{{ store.state.consoleErr }}</span>
        </div>
      </div>

      <div class="pt-2 pb-2" v-if="!store.state.ffFileOk && store.state.consoleErr">
        <span class="absolute transition-all duration-150">⚠</span>
        <span class="ml-8">{{ store.state.consoleErr }}</span>
      </div>
      <div class="pt-6 pb-2" v-if="store.state.consoleMsg">
        <span class="absolute transition-all duration-150"
          :class="store.state.ffProgress < 100 ? 'animate-bounce' : ''">⏳</span>
        <span class="ml-8">{{ store.state.consoleMsg }}</span>
        <span class="ml-2" v-if="store.state.ffProgress > 0">{{ store.state.ffProgress }}%</span>
      </div>
      <div class="pt-6 pb-2" v-if="video">
        <div class="pb-2 font-bold">🎊 Video Compressed 🎊</div>
        <video v-if="video" :src="video" class="rounded shadow cursor-pointer" controls />
        <div class="text-sm text-gray-300 pt-2">
          <b>To save:</b>&ensp;Right-click/long-press the video and click
          Save Video
        </div>
      </div>
    </div>
  </div>
  <div class="grid grid-cols-1 md:grid-cols-3 gap-16 pt-20">
    <div class="flex flex-col items-center gap-6 w-full text-black dark:text-white">
      <div><svg width="34" height="30" viewBox="0 0 44 40" fill="none" xmlns="http://www.w3.org/2000/svg"
          class="fill-black dark:fill-white">
          <path
            d="M39.2804 6.5598H19.1943L17.666 2.73996C17.2239 1.63404 15.9913 0.799805 14.8004 0.799805H2.80039C1.47703 0.799805 0.400391 1.87644 0.400391 3.1998V31.0398C0.400391 33.4216 2.33815 35.3598 4.72039 35.3598H17.2973C17.7433 37.5481 19.6825 39.1998 22.0004 39.1998C24.3183 39.1998 26.2575 37.5481 26.7034 35.3598H39.2804C41.6626 35.3598 43.6004 33.4216 43.6004 31.0398V10.8798C43.6004 8.49804 41.6626 6.5598 39.2804 6.5598ZM2.80039 2.7198H14.8004C15.1988 2.7198 15.7354 3.08316 15.8837 3.45324L17.126 6.5598H4.72039C3.83287 6.5598 3.00727 6.82956 2.32039 7.29036V3.1998C2.32039 2.93964 2.54023 2.7198 2.80039 2.7198ZM17.2935 33.4398H4.72039C3.39703 33.4398 2.32039 32.3632 2.32039 31.0398V10.8798C2.32039 9.55645 3.39703 8.4798 4.72039 8.4798H21.0404V9.43981H19.1204V11.3598H21.0404V13.2798H19.1204V15.1998H21.0404V17.1198H19.1204V19.0398H21.0404V20.9598H19.1204V22.8798H21.0404V23.8398H19.1204C18.3265 23.8398 17.6804 24.4859 17.6804 25.2798V27.1998C17.6804 27.8512 18.1177 28.3964 18.7124 28.5736C18.2309 29.8864 17.5397 31.9278 17.2935 33.4398ZM24.4004 26.7198H19.6004V25.7598H24.4004V26.7198ZM22.0004 37.2798C20.4121 37.2798 19.1204 35.9881 19.1204 34.3998C19.1204 33.1604 20.1356 30.2257 20.737 28.6398H23.2637C23.8652 30.2257 24.8804 33.1604 24.8804 34.3998C24.8804 35.9881 23.5887 37.2798 22.0004 37.2798ZM41.6804 31.0398C41.6804 32.3632 40.6037 33.4398 39.2804 33.4398H26.7073C26.461 31.9278 25.7698 29.8864 25.2884 28.5736C25.8831 28.3964 26.3204 27.8512 26.3204 27.1998V25.2798C26.3204 24.4859 25.6743 23.8398 24.8804 23.8398H22.9604V20.9598H24.8804V19.0398H22.9604V17.1198H24.8804V15.1998H22.9604V13.2798H24.8804V11.3598H22.9604V8.4798H39.2804C40.6037 8.4798 41.6804 9.55645 41.6804 10.8798V31.0398ZM23.4404 34.3998C23.4404 35.1952 22.7962 35.8398 22.0004 35.8398C21.2045 35.8398 20.5604 35.1952 20.5604 34.3998C20.5604 33.6044 21.2045 32.9598 22.0004 32.9598C22.7962 32.9598 23.4404 33.6044 23.4404 34.3998Z">
          </path>
        </svg></div>
      <h2 class="font-semibold text-center"><span><span class="false">Perfect Quality</span></span></h2>
      <p class="text-center"><span><span class="false">We use artificial intelligence to select best choices to compress
            your video without losing quality of video and audio.</span></span></p>
    </div>
    <!-- Add similar div blocks for other icons -->
    <div class="flex flex-col items-center items center gap-6 w-full text-black dark:text-white">
      <div><svg width="20" height="30" viewBox="0 0 20 34" fill="none" xmlns="http://www.w3.org/2000/svg"
          class="fill-black dark:fill-white">
          <path
            d="M18.8002 13.1842H12.4482L16.7402 1.14417C16.7828 1.02456 16.7963 0.89654 16.7795 0.770692C16.7628 0.644844 16.7164 0.524782 16.6441 0.42043C16.5718 0.316078 16.4757 0.230431 16.3637 0.17057C16.2518 0.110709 16.1272 0.0783511 16.0002 0.0761719H6.40023C6.22884 0.0765193 6.06209 0.131901 5.92455 0.234159C5.787 0.336417 5.68594 0.480145 5.63623 0.644172L0.436235 17.7722C0.400053 17.8916 0.3923 18.0178 0.413594 18.1407C0.434889 18.2637 0.484642 18.3799 0.558877 18.4802C0.633112 18.5805 0.729772 18.662 0.841133 18.7182C0.952495 18.7745 1.07547 18.8039 1.20023 18.8042H7.08023L4.44023 32.9762C4.40679 33.1556 4.43568 33.341 4.52213 33.5017C4.60858 33.6625 4.74738 33.7888 4.9155 33.8598C5.08363 33.9308 5.27097 33.9421 5.44645 33.892C5.62192 33.8419 5.77498 33.7333 5.88023 33.5842L19.4482 14.4482C19.5332 14.3288 19.5838 14.1883 19.5944 14.0421C19.6051 13.8959 19.5753 13.7496 19.5084 13.6192C19.4416 13.4888 19.3402 13.3792 19.2153 13.3025C19.0904 13.2258 18.9468 13.1848 18.8002 13.1842ZM6.68023 29.6882L8.83223 18.1522C8.85423 18.0355 8.84995 17.9153 8.81972 17.8004C8.78949 17.6856 8.73406 17.5789 8.65746 17.4881C8.58086 17.3973 8.48501 17.3248 8.37686 17.2756C8.26872 17.2265 8.15099 17.2021 8.03223 17.2042H2.28423L6.99223 1.67617H14.8562L10.5602 13.7162C10.5174 13.8367 10.504 13.9658 10.5214 14.0926C10.5387 14.2194 10.5862 14.3401 10.6599 14.4448C10.7336 14.5494 10.8312 14.6348 10.9448 14.6939C11.0583 14.7529 11.1843 14.7839 11.3122 14.7842H17.2482L6.68023 29.6882Z">
          </path>
        </svg></div>
      <h2 class="font-semibold text-center"><span><span class="false">Lightning
            Fast</span></span></h2>
      <p class="text-center"><span><span class="false">Reduce video size online, effective and faster than any other tools in
            competition for free.</span></span></p>
    </div>
    <div class="flex flex-col items-center items center gap-6 w-full text-black dark:text-white">
      <div><svg width="30" height="30" viewBox="0 0 34 35" xmlns="http://www.w3.org/2000/svg"
          class="fill-black dark:fill-white">
          <path
            d="M24.3883 16.5702L33.2478 7.44841C33.4862 7.20373 33.6754 6.91303 33.8045 6.59295C33.9336 6.27288 34 5.92972 34 5.58316C34 5.2366 33.9336 4.89345 33.8045 4.57337C33.6754 4.25329 33.4862 3.96259 33.2478 3.71792L30.3858 0.772793C30.148 0.527839 29.8656 0.333524 29.5548 0.200948C29.244 0.0683724 28.9109 0.000134531 28.5745 0.000134531C28.2381 0.000134531 27.905 0.0683724 27.5942 0.200948C27.2834 0.333524 27.0011 0.527839 26.7632 0.772793L17.9037 9.89455L8.51456 0.228789C8.44426 0.156265 8.36076 0.0987252 8.26883 0.0594664C8.17691 0.0202077 8.07836 0 7.97884 0C7.87931 0 7.78077 0.0202077 7.68884 0.0594664C7.59691 0.0987252 7.51341 0.156265 7.44311 0.228789L0.224785 7.6585C0.0830969 7.80499 0.00354744 8.00329 0.00354744 8.21001C0.00354744 8.41673 0.0830969 8.61503 0.224785 8.76152L9.61395 18.4273L3.85338 24.3575L3.66145 24.6539L0.0571425 33.9282C0.00949484 34.0454 -0.00880095 34.1729 0.00393485 34.2992C0.0166706 34.4256 0.0600295 34.5465 0.13003 34.6511C0.198769 34.7574 0.291712 34.8447 0.400756 34.9055C0.509801 34.9663 0.631638 34.9988 0.755651 35C0.853999 34.9996 0.951344 34.9796 1.04234 34.9412L10.0391 31.237L10.3295 31.0432L16.0985 25.1004L25.4877 34.7661C25.558 34.8387 25.6415 34.8962 25.7334 34.9355C25.8254 34.9747 25.9239 34.9949 26.0234 34.9949C26.123 34.9949 26.2215 34.9747 26.3134 34.9355C26.4054 34.8962 26.4889 34.8387 26.5592 34.7661L33.7775 27.3352C33.9194 27.1891 33.9991 26.9909 33.9991 26.7843C33.9991 26.5777 33.9194 26.3795 33.7775 26.2334L24.3883 16.5702ZM32.1776 6.34164L29.6338 8.96786L25.2921 4.49703L27.8359 1.87831C27.933 1.77825 28.0484 1.69888 28.1753 1.64473C28.3023 1.59058 28.4383 1.56271 28.5757 1.56271C28.7131 1.56271 28.8492 1.59058 28.9761 1.64473C29.1031 1.69888 29.2184 1.77825 29.3155 1.87831L32.1788 4.82468C32.2762 4.9247 32.3534 5.04351 32.4061 5.1743C32.4589 5.3051 32.486 5.44531 32.486 5.58691C32.486 5.72852 32.4589 5.86873 32.4061 5.99952C32.3534 6.13032 32.2762 6.24913 32.1788 6.34914L32.1776 6.34164ZM28.5611 10.0684L9.78281 29.3999L5.44112 24.9291L24.2195 5.59754L28.5611 10.0684ZM1.83561 8.21001L7.98248 1.88206L10.5202 4.49453L7.36172 7.74604L8.43317 8.84906L11.5916 5.59754L14.1294 8.21001L12.7749 9.60441L13.8451 10.7074L15.2008 9.31302L16.8359 10.9963L10.683 17.3243L1.83561 8.21001ZM8.43924 30.2203L2.1223 32.8202L4.64908 26.3172L8.43924 30.2203ZM32.174 26.7899L26.0271 33.1166L17.17 24.0011L23.3169 17.6719L24.952 19.3565L21.7935 22.608L22.8637 23.7097L26.0222 20.4582L28.5599 23.0707L27.2042 24.4663L28.2757 25.5694L29.6338 24.1737L32.174 26.7899Z">
          </path>
        </svg></div>
      <h2 class="font-semibold text-center"><span><span class="false">Easy To
            Use</span></span></h2>
      <p class="text-center"><span><span class="false">Simply upload your video and compress. It&#x27;s as easy as
            that!</span></span></p>
    </div>
    <div class="flex flex-col items-center items center gap-6 w-full text-black dark:text-white">
      <div><svg width="26" height="30" viewBox="0 0 34 40" fill="none" xmlns="http://www.w3.org/2000/svg"
          class="fill-black dark:fill-white">
          <path
            d="M17.0002 0.400391C16.5584 0.400391 16.2002 0.758551 16.2002 1.20039V4.40039C16.2002 4.84223 16.5584 5.20039 17.0002 5.20039C17.442 5.20039 17.8002 4.84223 17.8002 4.40039V1.20039C17.8002 0.758551 17.442 0.400391 17.0002 0.400391ZM5.68772 5.08789C5.48291 5.08789 5.28144 5.16915 5.1252 5.32539C4.8128 5.63779 4.8128 6.13795 5.1252 6.45039L7.38771 8.71289C7.7 9.02529 8.20032 9.02529 8.51268 8.71289C8.82516 8.40049 8.82516 7.90033 8.51268 7.58789L6.2502 5.32539C6.094 5.16915 5.89235 5.08789 5.68772 5.08789ZM28.3127 5.08789C28.108 5.08789 27.9064 5.16915 27.7502 5.32539L25.4877 7.58789C25.1752 7.90033 25.1752 8.40049 25.4877 8.71289C25.8001 9.02529 26.3004 9.02529 26.6127 8.71289L28.8752 6.45039C29.1876 6.13795 29.1876 5.63779 28.8752 5.32539C28.719 5.16915 28.5175 5.08789 28.3127 5.08789ZM17.0002 6.80039C11.7084 6.80039 6.60019 10.9145 6.60019 16.7379C6.60019 20.4103 8.02968 22.7442 9.35019 24.6129C10.6707 26.4815 11.8002 27.8543 11.8002 30.0004V34.8004C11.8002 36.1581 12.9259 37.2004 14.2002 37.2004H14.6002C14.6002 38.5161 15.6844 39.6004 17.0002 39.6004C18.316 39.6004 19.4002 38.5161 19.4002 37.2004H19.8002C21.0745 37.2004 22.2002 36.1581 22.2002 34.8004V30.0004C22.2002 27.8543 23.3297 26.4815 24.6502 24.6129C25.9707 22.7442 27.4002 20.4103 27.4002 16.7379C27.4002 10.9145 22.292 6.80039 17.0002 6.80039ZM17.0002 8.40039C21.4645 8.40039 25.8002 11.8554 25.8002 16.7379C25.8002 20.011 24.6297 21.8773 23.3502 23.6879C22.2013 25.3137 20.9042 26.9229 20.6502 29.2004H13.3502C13.0962 26.9229 11.7991 25.3137 10.6502 23.6879C9.37072 21.8773 8.20019 20.011 8.20019 16.7379C8.20019 11.8554 12.5359 8.40039 17.0002 8.40039ZM14.6502 10.0379C14.5406 10.0383 14.4289 10.0534 14.3127 10.0879C11.8402 11.0203 9.99795 13.1899 9.5252 15.8504C9.44991 16.2627 9.76283 16.7126 10.1752 16.7879C10.5875 16.8632 11.0249 16.5502 11.1002 16.1379C11.4736 14.0364 12.9384 12.3104 14.8877 11.5754C15.2352 11.4462 15.465 11.053 15.4002 10.6879C15.2675 10.2065 14.979 10.0362 14.6502 10.0379ZM1.0002 16.4004C0.558356 16.4004 0.200195 16.7585 0.200195 17.2004C0.200195 17.6422 0.558356 18.0004 1.0002 18.0004H4.2002C4.64204 18.0004 5.0002 17.6422 5.0002 17.2004C5.0002 16.7585 4.64204 16.4004 4.2002 16.4004H1.0002ZM29.8002 16.4004C29.3584 16.4004 29.0002 16.7585 29.0002 17.2004C29.0002 17.6422 29.3584 18.0004 29.8002 18.0004H33.0002C33.442 18.0004 33.8002 17.6422 33.8002 17.2004C33.8002 16.7585 33.442 16.4004 33.0002 16.4004H29.8002ZM13.4002 30.8004H20.6002V32.4004H13.4002V30.8004ZM13.4002 34.0004H20.6002V34.8004C20.6002 35.2811 20.2143 35.6004 19.8002 35.6004H14.2002C13.7861 35.6004 13.4002 35.2811 13.4002 34.8004V34.0004ZM16.2002 37.2004H17.8002C17.8002 37.6574 17.4572 38.0004 17.0002 38.0004C16.5432 38.0004 16.2002 37.6574 16.2002 37.2004Z">
          </path>
        </svg></div>
      <h2 class="font-semibold text-center"><span><span class="false">Works
            Anywhere</span></span></h2>
      <p class="text-center"><span><span class="false">FreeOnlineVideoCompressor.com is browser-based solution
            so no software to install. It works on any platform (Windows, Linux,
            Mac, Android, IOS).</span></span></p>
    </div>
    <div class="flex flex-col items-center items center gap-6 w-full text-black dark:text-white">
      <div><svg width="30" height="30" viewBox="0 0 40 40" fill="none" xmlns="http://www.w3.org/2000/svg"
          class="fill-black dark:fill-white">
          <path
            d="M33.2245 5.02563L20.2246 2.02565C20.0768 1.99145 19.9232 1.99145 19.7754 2.02565L6.77551 5.02563C6.55509 5.07643 6.3584 5.20047 6.21757 5.37748C6.07674 5.55449 6.00008 5.77402 6.00011 6.00022V20.6593C5.98572 24.6817 7.34258 28.589 9.84695 31.7367C12.3513 34.8844 15.8537 37.0846 19.7764 37.9746C19.9242 38.0085 20.0778 38.0085 20.2256 37.9746C24.148 37.0843 27.6499 34.8839 30.1539 31.7362C32.6579 28.5885 34.0145 24.6815 33.9999 20.6593V6.00022C33.9999 5.77402 33.9233 5.55449 33.7824 5.37748C33.6416 5.20047 33.4449 5.07643 33.2245 5.02563ZM31.9999 20.6593C32.013 24.1893 30.8359 27.6206 28.6585 30.3991C26.4811 33.1776 23.4307 35.141 20 35.9722C16.5693 35.141 13.5189 33.1776 11.3415 30.3991C9.16414 27.6206 7.98696 24.1893 8.0001 20.6593V6.79562L20 4.02663L31.9999 6.79562V20.6593Z">
          </path>
          <path
            d="M15.5804 17.9676C15.5034 17.8886 15.4115 17.8257 15.3099 17.7825C15.2084 17.7393 15.0993 17.7167 14.989 17.716C14.8787 17.7153 14.7693 17.7365 14.6673 17.7784C14.5652 17.8203 14.4725 17.8821 14.3945 17.9601C14.3165 18.0381 14.2547 18.1308 14.2128 18.2329C14.1709 18.3349 14.1497 18.4443 14.1504 18.5546C14.1511 18.6649 14.1737 18.774 14.2169 18.8755C14.2601 18.9771 14.323 19.069 14.402 19.146L17.938 22.6816C18.0943 22.8379 18.3062 22.9256 18.5272 22.9256C18.7482 22.9256 18.9601 22.8379 19.1164 22.6816L25.5983 16.2005C25.6773 16.1235 25.7402 16.0316 25.7834 15.93C25.8266 15.8285 25.8492 15.7194 25.8499 15.6091C25.8506 15.4988 25.8294 15.3894 25.7875 15.2874C25.7456 15.1853 25.6838 15.0926 25.6058 15.0146C25.5278 14.9366 25.4351 14.8748 25.333 14.8329C25.231 14.791 25.1216 14.7698 25.0113 14.7705C24.901 14.7712 24.7919 14.7938 24.6904 14.837C24.5889 14.8802 24.4969 14.9431 24.4199 15.0221L18.5272 20.9144L15.5804 17.9676Z">
          </path>
        </svg></div>
      <h2 class="font-semibold text-center"><span><span class="false">Privacy
            Guaranteed</span></span></h2>
      <p class="text-center"><span><span class="false">Your videos are uploaded via a
            secure 256-bit encrypted SSL connection and we didn't store user's videos like other
            competitors in
            market.</span></span></p>
    </div>
    <div class="flex flex-col items-center items center gap-6 w-full text-black dark:text-white">
      <div><svg width="38" height="30" viewBox="0 0 40 34" fill="none" xmlns="http://www.w3.org/2000/svg"
          class="fill-black dark:fill-white">
          <path
            d="M20.0034 33.9994C19.7205 33.9994 19.4486 33.8872 19.2469 33.6871L18.9997 33.4399C15.2928 29.7361 12.4885 27.1282 10.0137 24.8263C9.26926 24.1338 8.55051 23.4643 7.84014 22.7969C5.1731 20.2865 2.6868 17.7907 1.55995 14.8602C0.204692 11.3293 0.59131 7.61187 2.62079 4.65933C4.62566 1.74135 7.93863 0 11.4826 0C14.3995 0 17.2793 1.163 19.8101 3.36431L19.9992 3.54976L20.1396 3.41041C22.7207 1.163 25.6 0 28.5169 0C32.0604 0 35.3733 1.74135 37.3787 4.65933C39.4082 7.61187 39.7948 11.3303 38.4401 14.8591C37.3138 17.7876 34.8274 20.2844 32.1609 22.7958C31.4506 23.4632 30.7329 24.1306 29.9911 24.8211C27.5152 27.124 24.7083 29.734 20.9998 33.4399C20.9249 33.5153 20.8626 33.5771 20.795 33.6505C20.598 33.8663 20.3209 33.992 20.0291 33.9994C20.0207 33.9994 20.0118 33.9994 20.0034 33.9994ZM11.4826 2.14578C8.64533 2.14578 5.99349 3.53929 4.38886 5.87471C2.76591 8.23633 2.46468 11.2308 3.56272 14.0901C4.52769 16.5984 6.83168 18.9003 9.31013 21.2336C10.0174 21.8989 10.7335 22.5653 11.4753 23.2547C13.8469 25.4613 16.5202 27.9476 19.9998 31.4072C23.4877 27.9392 26.1594 25.455 28.5294 23.2505C29.2692 22.5622 29.9837 21.8968 30.691 21.2336C33.1678 18.8992 35.4729 16.5963 36.4368 14.089C37.5348 11.2308 37.2331 8.23633 35.6101 5.87471C34.005 3.53929 31.3542 2.14578 28.5169 2.14578C26.1249 2.14578 23.7329 3.12752 21.5976 4.98413L20.7573 5.81708C20.3403 6.22675 19.6708 6.2299 19.2527 5.82023L18.3553 4.94117C16.2672 3.12752 13.8746 2.14578 11.4826 2.14578Z">
          </path>
        </svg></div>
      <h2 class="font-semibold text-center"><span><span class="false">It&#x27;s
            Free</span></span></h2>
      <p class="text-center"><span><span class="false">Since 2023 we have compressed millions
            of videos for free! There is no software to install, registrations, or
            watermarks.</span></span></p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import { store } from "../store/index";

import VideoService from "../common/video.service";

const inputFile = ref<File | null>(null);
const video = ref<string | null>(null);

let targetSize = ref(25);
let targetScale = ref(1.0);
let targetScaleAuto = ref(false);
let resScale = ref([1.0, 0.75, 0.5, 0.25]);
let minimumSize = ref(0);

let vs: VideoService;

// onMounted - initializes ffmpeg
onMounted(() => {
  vs = new VideoService();
});

// onFileChanged - sets the inputFile to the file passed in
const onFileChanged = async (event: Event) => {
  const target = event.target as HTMLInputElement;

  if (!target.files || !targetSize.value) {
    return store.commit("consoleErr", "No file selected");
  }
  store.commit("consoleErr", "");
  store.commit("ffFileOkPre", true);
  inputFile.value = target.files[0];
  store.commit("ffFilename", target.files[0].name);
  store.commit("ffFilesize", target.files[0].size);

  // Adjust targetSize if targetSize > originalSize (bytes -> MB)
  if (1 > Number(target.files[0].size) / 1024 / 1024) {
    // If originalSize is smaller than 1 MB, just use the originalSize
    targetSize.value = Number(target.files[0].size) / 1024 / 1024;
  } else if (8 > Number(target.files[0].size) / 1024 / 1024) {
    // If originalSize is smaller than 8 MB, use Math.floor()
    targetSize.value = Math.floor(Number(target.files[0].size) / 1024 / 1024);
  } else if (targetSize.value > Number(target.files[0].size) / 1024 / 1024) {
    // Reset to 8 MB for all videos larger than 8 MB
    targetSize.value = 8;
  };

  // Adjust targetScale if ratio between targetSize and originalSize is too big
  let targetScaleRatio = (Number(targetSize.value) * 1.5) / (Number(target.files[0].size) / 1024 / 1024);
  let targetScaleRecc = resScale.value.reduce(function (upper, lower) {
    return (Math.min(lower, targetScaleRatio) != targetScaleRatio ? upper : lower);
  });
  targetScaleRecc != 1.0 ? targetScaleAuto.value = true : targetScaleAuto.value = false;
  targetScale.value = targetScaleRecc;

  try {
    minimumSize.value = (await vs.getMinimumSize(
      target.files[0],
      targetSize.value
    )) as number;
  } catch {
    // Likely no audio if this failed
    minimumSize.value = 0
  }
  store.commit("ffFileOkPre", false);
};

// onSubmit - called when submit button is pressed245
// reads the file and calls the ffmpeg function
const onSubmit = async () => {
  if (!inputFile.value) {
    return store.commit("consoleErr", "No file selected");
  };
  if (store.state.ffFilesize < Number(targetSize.value) * 1024 * 1024) {
    return store.commit("consoleErr", "File is smaller than your target size!");
  };
  console.log(targetSize.value, Number(minimumSize.value) / 1024 / 1024);
  if (targetSize.value < Number(minimumSize.value) / 1024 / 1024) {
    return store.commit("consoleErr", "Target size is too small!");
  };

  store.commit("ffFileOk", true);

  const renamedFile = new File([inputFile.value], "input.mp4");

  const videoData = await vs.shrinkVideo(
    renamedFile,
    targetSize.value,
    targetScale.value
  );

  if (!videoData) return;
  video.value = URL.createObjectURL(
    new Blob([videoData.buffer], { type: "video/mp4" })
  );
};
</script>

<script lang="ts">
export default {
  data() {
    return {
    };
  },
  methods: {
    reload() {
      window.location.reload();
    },
    sizeBytes(x: number) {
      // https://stackoverflow.com/a/39906526/15923512 CC BY-SA 4.0
      const units = ["bytes", "KB", "MB", "GB", "TB", "PB", "EB", "ZB", "YB"];
      let l = 0,
        n = x || 0;
      while (n >= 1024 && ++l) {
        n = n / 1024;
      }
      return n.toFixed(n < 10 && l > 0 ? 2 : 0) + " " + units[l];
    },
  },
};
</script>
