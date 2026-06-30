<template>
    <main class="app-shell">
        <section class="workspace">
            <aside class="brand-panel">
                <a class="brand-mark" href="https://github.com/SubConv/SubConv" target="_blank" rel="noreferrer">
                    <span class="brand-orbit">
                        <span class="brand-core">S</span>
                    </span>
                    <span>
                        <span class="brand-name">SubConv</span>
                        <span class="brand-link">
                            <i class="fa-brands fa-github"></i>
                            GitHub
                        </span>
                    </span>
                </a>

                <div class="signal-stack" aria-hidden="true">
                    <span></span>
                    <span></span>
                    <span></span>
                </div>

                <div class="metrics">
                    <div class="metric">
                        <span>模板</span>
                        <strong>{{ selectedTemplate || '...' }}</strong>
                    </div>
                    <div class="metric">
                        <span>规则代理</span>
                        <strong>{{ proxy_switch ? 'ON' : 'OFF' }}</strong>
                    </div>
                    <div class="metric">
                        <span>备用节点</span>
                        <strong>{{ standby_switch ? 'ON' : 'OFF' }}</strong>
                    </div>
                </div>
            </aside>

            <section class="control-panel">
                <header class="panel-header">
                    <div>
                        <p class="eyebrow">Mihomo Config Generator</p>
                        <h1>订阅转换控制台</h1>
                    </div>
                    <div class="status-pill" :class="{ danger: hasRuntimeConfigError }">
                        <span class="status-dot"></span>
                        {{ runtimeStatusText }}
                    </div>
                </header>

                <el-form label-position="top" class="main-form">
                    <div class="form-grid">
                        <el-form-item label="订阅">
                            <el-input
                                type="textarea"
                                v-model="linkInput"
                                :rows="6"
                                resize="none"
                                placeholder="请粘贴订阅链接，或者分享链接，多个订阅链接请换行或用 | 符号隔开"
                            ></el-input>
                        </el-form-item>

                        <div class="settings-column">
                            <el-form-item label="模板">
                                <el-select
                                    v-model="selectedTemplate"
                                    class="template-select"
                                    :disabled="!selectedTemplate"
                                    :loading="isLoadingRuntimeConfig"
                                    :placeholder="templatePlaceholder"
                                >
                                    <el-option
                                        v-for="template in templateOptions"
                                        :key="template.value"
                                        :label="template.label"
                                        :value="template.value"
                                    ></el-option>
                                </el-select>
                            </el-form-item>

                            <el-form-item label="更新间隔">
                                <div class="interval-row">
                                    <el-input v-model="time" placeholder="1800"></el-input>
                                    <span>秒</span>
                                </div>
                            </el-form-item>

                            <div class="toggle-card">
                                <div>
                                    <strong>代理规则集</strong>
                                    <span>关闭后将直接从 GitHub 获取规则集</span>
                                </div>
                                <el-switch v-model="proxy_switch"></el-switch>
                            </div>

                            <div class="toggle-card">
                                <div>
                                    <strong>备用节点</strong>
                                    <span>只出现在手动选择分组</span>
                                </div>
                                <el-switch v-model="standby_switch"></el-switch>
                            </div>
                        </div>
                    </div>

                    <transition name="slide-fade">
                        <el-form-item v-if="standby_switch" label="备用节点">
                            <el-input
                                type="textarea"
                                v-model="standby"
                                :rows="4"
                                resize="none"
                                placeholder="请粘贴备用节点，多个备用节点请换行或用 | 符号隔开"
                            ></el-input>
                        </el-form-item>
                    </transition>

                    <el-form-item label="新订阅链接">
                        <el-input
                            type="textarea"
                            v-model="linkOutput"
                            :rows="3"
                            resize="none"
                            readonly
                            placeholder="生成后的订阅链接会显示在这里"
                        ></el-input>
                    </el-form-item>

                    <div class="actions">
                        <el-button type="primary" class="generate-button" @click="submitForm">
                            <i class="fa fa-bolt" aria-hidden="true"></i>
                            生成
                        </el-button>
                        <el-button class="copy-button" @click="copyForm">
                            <i class="fa fa-copy" aria-hidden="true"></i>
                            复制
                        </el-button>
                    </div>
                </el-form>
            </section>
        </section>

        <footer class="footer">
            <a href="https://github.com/SubConv/SubConv" target="_blank" rel="noreferrer">
                <i class="fa fa-link" aria-hidden="true"></i>
                API 后端项目: SubConv
            </a>
            <a href="https://github.com/musanico" target="_blank" rel="noreferrer">
                <i class="fa fa-pencil" aria-hidden="true"></i>
                UI designed by @Musanico
            </a>
        </footer>
    </main>
</template>

<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import { ElButton, ElInput, ElForm, ElFormItem, ElSwitch, ElMessage, ElSelect, ElOption } from 'element-plus'
import 'element-plus/es/components/button/style/css'
import 'element-plus/es/components/input/style/css'
import 'element-plus/es/components/form/style/css'
import 'element-plus/es/components/form-item/style/css'
import 'element-plus/es/components/switch/style/css'
import 'element-plus/es/components/select/style/css'
import 'element-plus/es/components/option/style/css'
import 'element-plus/es/components/message/style/css'

const linkInput = ref('')
const linkOutput = ref('')
const time = ref('')
const standby = ref('')
const defaultTemplate = ref<string | null>(null)
const selectedTemplate = ref<string | null>(null)
const availableTemplates = ref<string[]>([])
const isLoadingRuntimeConfig = ref(true)
const hasRuntimeConfigError = ref(false)
const standby_switch = ref(false)
const proxy_switch = ref(true)

const templateOptions = computed(() => [
    ...availableTemplates.value.map((templateName) => ({
        value: templateName,
        label: templateName
    }))
])

const templatePlaceholder = computed(() => {
    if (isLoadingRuntimeConfig.value) {
        return '模板配置加载中...'
    }
    if (hasRuntimeConfigError.value) {
        return '模板配置加载失败，请刷新网页后重试'
    }
    return '请选择模板'
})

const runtimeStatusText = computed(() => {
    if (isLoadingRuntimeConfig.value) {
        return '同步模板'
    }
    if (hasRuntimeConfigError.value) {
        return '配置异常'
    }
    return '系统就绪'
})

const sleep = (ms: number) => new Promise((resolve) => window.setTimeout(resolve, ms))

const initializeTemplateSelection = async () => {
    const maxAttempts = 3
    const retryDelayMs = [500, 1500]

    isLoadingRuntimeConfig.value = true
    hasRuntimeConfigError.value = false

    for (let attempt = 1; attempt <= maxAttempts; attempt += 1) {
        try {
            const response = await fetch('/config')
            if (!response.ok) {
                throw new Error(`Failed to load runtime config: ${response.status}`)
            }

            const runtimeConfig = await response.json() as {
                defaultTemplate?: string,
                availableTemplates?: string[]
            }

            const runtimeTemplates = Array.isArray(runtimeConfig.availableTemplates)
                ? runtimeConfig.availableTemplates
                : []

            if (runtimeTemplates.length === 0) {
                throw new Error('Runtime config did not include any available templates.')
            }

            availableTemplates.value = runtimeTemplates
            defaultTemplate.value = typeof runtimeConfig.defaultTemplate === 'string'
                ? runtimeConfig.defaultTemplate
                : null
            selectedTemplate.value = runtimeTemplates.includes(defaultTemplate.value ?? '')
                ? defaultTemplate.value
                : runtimeTemplates[0]
            isLoadingRuntimeConfig.value = false
            return
        }
        catch (error) {
            console.error(`Failed to initialize template selection from runtime config (attempt ${attempt}/${maxAttempts}).`, error)

            if (attempt < maxAttempts) {
                await sleep(retryDelayMs[attempt - 1])
                continue
            }

            availableTemplates.value = []
            defaultTemplate.value = null
            selectedTemplate.value = null
            hasRuntimeConfigError.value = true
            isLoadingRuntimeConfig.value = false
            ElMessage({
                message: '模板配置加载失败，请刷新网页后重试',
                type: 'error'
            })
        }
    }
}

onMounted(async () => {
    await initializeTemplateSelection()
})

const submitForm = () => {
    let result = window.location.protocol + '//' + window.location.host
    if (linkInput.value !== '') {
        if (!selectedTemplate.value) {
            ElMessage({
                message: '模板配置加载失败，请刷新网页后重试',
                type: 'error'
            })
            linkOutput.value = ''
            return false
        }
        result += '/sub?url=' + encodeURIComponent(linkInput.value)
        result += '&template=' + encodeURIComponent(selectedTemplate.value)
        if (time.value !== '') {
            if (/^[1-9][0-9]*$/.test(time.value)) {
                result += '&interval=' + time.value
            }
            else {
                ElMessage({
                    message: '时间间隔必须为整数',
                    type: 'error'
                })
                linkOutput.value = ''
                return false
            }
        }
        if (standby_switch.value) {
            if (standby.value !== '') {
                result += '&urlstandby=' + encodeURIComponent(standby.value)
            }
        }
        if (!proxy_switch.value) {
            result += '&npr=1'
        }
    } else {
        ElMessage({
            message: '订阅链接不能为空',
            type: 'error'
        })
        linkOutput.value = ''
        return false
    }
    linkOutput.value = result
}

const copyForm = () => {
    if (linkOutput.value === '') {
        ElMessage({
            message: '请先生成订阅链接',
            type: 'warning'
        })
        return
    }

    navigator.clipboard.writeText(linkOutput.value)
    ElMessage({
        message: '复制成功',
        type: 'success'
    })
}
</script>

<style scoped>
:global(*) {
    box-sizing: border-box;
}

:global(body) {
    min-width: 320px;
    margin: 0;
    background: #080b12;
    color: #eef4ff;
    font-family: Inter, "PingFang SC", "Microsoft YaHei", Arial, sans-serif;
}

:global(#app) {
    min-height: 100vh;
}

.app-shell {
    position: relative;
    min-height: 100vh;
    overflow: hidden;
    padding: 48px 24px 28px;
    background:
        radial-gradient(circle at 12% 18%, rgba(0, 213, 255, 0.24), transparent 26%),
        radial-gradient(circle at 86% 10%, rgba(255, 63, 160, 0.2), transparent 28%),
        linear-gradient(135deg, #090d16 0%, #111827 48%, #071019 100%);
}

.app-shell::before {
    position: absolute;
    inset: 0;
    pointer-events: none;
    content: "";
    background-image:
        linear-gradient(rgba(255, 255, 255, 0.05) 1px, transparent 1px),
        linear-gradient(90deg, rgba(255, 255, 255, 0.05) 1px, transparent 1px);
    background-size: 42px 42px;
    mask-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.7), transparent 82%);
}

.workspace {
    position: relative;
    z-index: 1;
    display: grid;
    grid-template-columns: minmax(240px, 320px) minmax(0, 820px);
    gap: 18px;
    max-width: 1180px;
    margin: 0 auto;
}

.brand-panel,
.control-panel {
    border: 1px solid rgba(154, 221, 255, 0.22);
    background: rgba(9, 15, 27, 0.78);
    box-shadow: 0 24px 80px rgba(0, 0, 0, 0.42);
    backdrop-filter: blur(20px);
}

.brand-panel {
    display: flex;
    flex-direction: column;
    min-height: 640px;
    padding: 26px;
    border-radius: 8px;
}

.brand-mark {
    display: flex;
    align-items: center;
    gap: 16px;
    color: #f8fbff;
    text-decoration: none;
}

.brand-orbit {
    position: relative;
    display: grid;
    width: 62px;
    height: 62px;
    place-items: center;
    border: 1px solid rgba(115, 226, 255, 0.46);
    border-radius: 50%;
    background: linear-gradient(135deg, rgba(0, 225, 255, 0.2), rgba(255, 62, 165, 0.18));
    box-shadow: 0 0 30px rgba(32, 218, 255, 0.24);
}

.brand-orbit::after {
    position: absolute;
    inset: -6px;
    border: 1px dashed rgba(129, 230, 217, 0.45);
    border-radius: 50%;
    content: "";
    animation: spin 16s linear infinite;
}

.brand-core {
    font-size: 28px;
    font-weight: 800;
}

.brand-name {
    display: block;
    font-size: 28px;
    font-weight: 800;
    line-height: 1;
    letter-spacing: 0;
}

.brand-link {
    display: flex;
    gap: 8px;
    align-items: center;
    margin-top: 8px;
    color: #8bd7ff;
    font-size: 13px;
}

.signal-stack {
    display: grid;
    gap: 14px;
    margin: 76px 0;
}

.signal-stack span {
    display: block;
    height: 8px;
    border-radius: 999px;
    background: linear-gradient(90deg, #26e7ff, #f557b6, transparent);
    box-shadow: 0 0 24px rgba(38, 231, 255, 0.34);
}

.signal-stack span:nth-child(2) {
    width: 76%;
    background: linear-gradient(90deg, #92f7c3, #26e7ff, transparent);
}

.signal-stack span:nth-child(3) {
    width: 54%;
    background: linear-gradient(90deg, #fcd34d, #f557b6, transparent);
}

.metrics {
    display: grid;
    gap: 12px;
    margin-top: auto;
}

.metric {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
    min-height: 58px;
    padding: 14px;
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 8px;
    background: rgba(255, 255, 255, 0.05);
}

.metric span {
    color: #91a3b8;
    font-size: 13px;
}

.metric strong {
    max-width: 145px;
    overflow: hidden;
    color: #f8fbff;
    font-size: 18px;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.control-panel {
    min-width: 0;
    padding: 30px;
    border-radius: 8px;
}

.panel-header {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: 20px;
    margin-bottom: 30px;
}

.eyebrow {
    margin: 0 0 8px;
    color: #65e4ff;
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 0.16em;
    text-transform: uppercase;
}

h1 {
    margin: 0;
    color: #f8fbff;
    font-size: clamp(32px, 5vw, 54px);
    line-height: 1.04;
    letter-spacing: 0;
}

.status-pill {
    display: inline-flex;
    flex: 0 0 auto;
    align-items: center;
    gap: 9px;
    min-height: 36px;
    padding: 0 14px;
    border: 1px solid rgba(93, 239, 190, 0.34);
    border-radius: 999px;
    color: #b7ffdd;
    background: rgba(23, 122, 84, 0.16);
    font-size: 13px;
    font-weight: 700;
}

.status-pill.danger {
    border-color: rgba(255, 97, 116, 0.42);
    color: #ffbdc7;
    background: rgba(128, 29, 45, 0.22);
}

.status-dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: currentColor;
    box-shadow: 0 0 14px currentColor;
}

.main-form {
    display: grid;
    gap: 20px;
}

.form-grid {
    display: grid;
    grid-template-columns: minmax(0, 1.45fr) minmax(240px, 0.85fr);
    gap: 20px;
}

.settings-column {
    display: grid;
    align-content: start;
    gap: 14px;
}

.template-select {
    width: 100%;
}

.interval-row {
    display: grid;
    grid-template-columns: minmax(0, 1fr) auto;
    gap: 10px;
    align-items: center;
    color: #9fb3c8;
}

.toggle-card {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 16px;
    min-height: 76px;
    padding: 16px;
    border: 1px solid rgba(148, 163, 184, 0.18);
    border-radius: 8px;
    background: linear-gradient(135deg, rgba(255, 255, 255, 0.075), rgba(255, 255, 255, 0.025));
}

.toggle-card strong,
.toggle-card span {
    display: block;
}

.toggle-card strong {
    color: #edf7ff;
    font-size: 15px;
}

.toggle-card span {
    margin-top: 5px;
    color: #8da2b9;
    font-size: 12px;
    line-height: 1.5;
}

.actions {
    display: flex;
    flex-wrap: wrap;
    gap: 12px;
    justify-content: flex-end;
}

.generate-button,
.copy-button {
    min-width: 132px;
    height: 44px;
    border-radius: 8px;
    font-weight: 800;
}

.generate-button {
    border: 0;
    background: linear-gradient(135deg, #10cfff, #7c3aed 54%, #f43f8f);
    box-shadow: 0 12px 34px rgba(16, 207, 255, 0.22);
}

.copy-button {
    border-color: rgba(137, 210, 255, 0.28);
    color: #dff5ff;
    background: rgba(255, 255, 255, 0.08);
}

.actions i {
    margin-right: 7px;
}

.footer {
    position: relative;
    z-index: 1;
    display: flex;
    flex-wrap: wrap;
    gap: 14px;
    justify-content: center;
    max-width: 1180px;
    margin: 20px auto 0;
}

.footer a {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    color: #91a3b8;
    font-size: 13px;
    text-decoration: none;
}

.footer a:hover {
    color: #dff7ff;
}

:deep(.el-form-item) {
    margin-bottom: 0;
}

:deep(.el-form-item__label) {
    margin-bottom: 9px;
    color: #b8c7d9;
    font-size: 13px;
    font-weight: 800;
}

:deep(.el-textarea__inner),
:deep(.el-input__wrapper),
:deep(.el-select__wrapper) {
    border: 1px solid rgba(143, 213, 255, 0.18);
    border-radius: 8px;
    color: #f7fbff;
    background: rgba(2, 8, 23, 0.72);
    box-shadow: inset 0 0 0 1px rgba(255, 255, 255, 0.02);
}

:deep(.el-textarea__inner) {
    min-height: 120px !important;
    padding: 14px;
    font-family: "SFMono-Regular", Consolas, "Liberation Mono", monospace;
    line-height: 1.55;
}

:deep(.el-textarea__inner:focus),
:deep(.el-input__wrapper.is-focus),
:deep(.el-select__wrapper.is-focused) {
    border-color: rgba(77, 221, 255, 0.74);
    box-shadow: 0 0 0 3px rgba(77, 221, 255, 0.14);
}

:deep(.el-input__inner),
:deep(.el-select__selected-item) {
    color: #f7fbff;
}

:deep(.el-input__inner::placeholder),
:deep(.el-textarea__inner::placeholder) {
    color: #60738b;
}

:deep(.el-switch.is-checked .el-switch__core) {
    border-color: #1fdcff;
    background: linear-gradient(90deg, #1fdcff, #7c3aed);
}

.slide-fade-enter-active,
.slide-fade-leave-active {
    transition: all 0.18s ease;
}

.slide-fade-enter-from,
.slide-fade-leave-to {
    opacity: 0;
    transform: translateY(-8px);
}

@keyframes spin {
    to {
        transform: rotate(360deg);
    }
}

@media (max-width: 900px) {
    .app-shell {
        padding: 24px 14px;
    }

    .workspace,
    .form-grid {
        grid-template-columns: 1fr;
    }

    .brand-panel {
        min-height: auto;
    }

    .signal-stack {
        margin: 28px 0;
    }

    .panel-header {
        flex-direction: column;
    }

    .actions {
        justify-content: stretch;
    }

    .generate-button,
    .copy-button {
        flex: 1 1 150px;
    }
}

@media (max-width: 560px) {
    .control-panel,
    .brand-panel {
        padding: 20px;
    }

    h1 {
        font-size: 34px;
    }

    .footer {
        justify-content: flex-start;
    }
}
</style>
