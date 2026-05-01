# 在线视频播放

这个小工具会把你粘贴的视频网页地址交给所选的第三方解析接口，并在当前页面生成播放窗口。它只做在线播放入口，不保存视频、不提供下载，也不绕过会员、登录、DRM 或平台权限限制。若下方播放窗口黑屏或被浏览器拦截，可以点击生成后的“打开解析页”到新页面观看。

<div class="video-tool" id="videoTool">
  <div class="video-tool__field">
    <label for="parseApi">解析接口</label>
    <select id="parseApi">
      <option value="https://jx.xmflv.com/?url=">XMFLV</option>
      <option value="https://jx.playerjy.com/?url=">PlayerJY</option>
      <option value="__custom__">自定义</option>
    </select>
  </div>

  <div class="video-tool__field" id="customApiWrap" hidden>
    <label for="customParseApi">自定义解析接口前缀</label>
    <input id="customParseApi" placeholder="例如：https://example.com/?url=" />
  </div>

  <div class="video-tool__grid">
    <div class="video-tool__field">
      <label for="parseUrl">视频网页地址</label>
      <input id="parseUrl" placeholder="粘贴 MGTV / 腾讯 / 爱奇艺 / B 站等网页链接" />
    </div>
    <button class="md-button md-button--primary video-tool__button" id="playBtn" type="button">播放</button>
  </div>

  <p class="video-tool__message" id="parseMessage"></p>
  <p class="video-tool__link-row">
    <a id="openParsePage" target="_blank" rel="noreferrer" hidden>打开解析页</a>
  </p>
  <iframe id="player" title="视频播放窗口" allowfullscreen allow="autoplay; encrypted-media" hidden></iframe>
</div>

<script>
  (function () {
    const parseApi = document.getElementById('parseApi')
    const customApiWrap = document.getElementById('customApiWrap')
    const customParseApi = document.getElementById('customParseApi')
    const parseUrl = document.getElementById('parseUrl')
    const playBtn = document.getElementById('playBtn')
    const player = document.getElementById('player')
    const parseMessage = document.getElementById('parseMessage')
    const openParsePage = document.getElementById('openParsePage')

    const savedApi = localStorage.getItem('videoToolParseApi') || parseApi.options[0].value
    const preset = Array.from(parseApi.options).some((option) => option.value === savedApi)

    if (preset) {
      parseApi.value = savedApi
    } else {
      parseApi.value = '__custom__'
      customParseApi.value = savedApi
    }

    handleParserChange()

    parseApi.addEventListener('change', handleParserChange)
    playBtn.addEventListener('click', playVideo)
    parseUrl.addEventListener('keydown', function (event) {
      if (event.key === 'Enter') playVideo()
    })

    function handleParserChange() {
      customApiWrap.hidden = parseApi.value !== '__custom__'
    }

    function assertUrl(url) {
      const parsed = new URL(url)
      if (!['http:', 'https:'].includes(parsed.protocol)) {
        throw new Error('只支持 http/https 链接')
      }
    }

    function playVideo() {
      const api = parseApi.value === '__custom__'
        ? customParseApi.value.trim()
        : parseApi.value.trim()
      const url = parseUrl.value.trim()

      try {
        if (!api) throw new Error('请先填写解析接口前缀')
        assertUrl(url)

        localStorage.setItem('videoToolParseApi', api)
        const parsedUrl = api + encodeURIComponent(url)
        player.src = parsedUrl
        player.hidden = false
        openParsePage.href = parsedUrl
        openParsePage.hidden = false
        parseMessage.textContent = '已生成播放地址。如果下方窗口黑屏或拒绝连接，可以点击“打开解析页”直接播放。'
      } catch (error) {
        player.hidden = true
        openParsePage.hidden = true
        parseMessage.textContent = error.message || '播放失败'
      }
    }
  })()
</script>
