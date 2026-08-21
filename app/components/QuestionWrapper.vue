<script setup lang="ts">
const props = defineProps<{
    language?: string
}>()

const langMap: Record<string, string> = {
    rs: 'rust', py: 'python', ts: 'typescript', tsx: 'typescript',
    js: 'javascript', jsx: 'javascript', mjs: 'javascript', cjs: 'javascript',
    sh: 'shell', zsh: 'shell', yml: 'yaml', md: 'markdown', html: 'xml', toml: 'toml',
}

const resolvedLang = computed(() => {
    const l = (props.language || '').toLowerCase()
    if (!l) return ''
    return langMap[l] || l
})

const bodyRef = ref<HTMLElement | null>(null)
const groupName = `q-${Math.random().toString(36).slice(2)}`

function findCorrectIndex(details: HTMLElement, optionEls: HTMLElement[]): number {
    const text = details.textContent || ''
    const match = text.match(/Answer:\s*([^\n]+)/i)
    if (!match) return -1

    const answer = match[1]!.trim()

    const letterMatch = answer.match(/^([A-D])\b/i)
    if (letterMatch) {
        return letterMatch[1]!.toUpperCase().charCodeAt(0) - 65
    }

    const answerHead = answer.split(/\s+[—-]\s+/)[0]!.trim().toLowerCase()
    return optionEls.findIndex((el) => {
        const optionText = (el.textContent || '').trim().toLowerCase()
        return optionText === answerHead || optionText.startsWith(answerHead)
    })
}

function setupOptions() {
    const root = bodyRef.value
    if (!root) return

    const checkboxes = Array.from(root.querySelectorAll<HTMLInputElement>('input[type="checkbox"]'))
    if (checkboxes.length === 0) return

    const optionItems = checkboxes.map((cb) => cb.closest('li')).filter((li): li is HTMLLIElement => !!li)
    if (optionItems.length !== checkboxes.length) return

    const details = root.querySelector('details')
    const correctIndex = details ? findCorrectIndex(details, optionItems) : -1

    checkboxes.forEach((checkbox, i) => {
        checkbox.type = 'radio'
        checkbox.name = groupName
        checkbox.removeAttribute('disabled')
        checkbox.checked = false

        const li = optionItems[i]!
        li.classList.add('quiz-option')

        const select = () => {
            checkbox.checked = true
            optionItems.forEach((el) => el.classList.remove('is-correct', 'is-incorrect'))

            if (correctIndex < 0) return

            if (i === correctIndex) {
                li.classList.add('is-correct')
            } else {
                li.classList.add('is-incorrect')
                optionItems[correctIndex]?.classList.add('is-correct')
            }
        }

        checkbox.addEventListener('change', select)
        li.addEventListener('click', (e) => {
            if (e.target !== checkbox) select()
        })
    })
}

onMounted(() => {
    nextTick(setupOptions)
})
</script>

<template>
    <div class="question-wrapper-parent p-4">
        <div class="question-wrapper">
            <div class="question-header">
                <div class="flex items-center gap-2">
                    <span class="question-dot" />
                    <span class="question-dot" />
                    <span class="question-dot" />
                </div>
                <div class="flex items-center gap-3">
                    <span v-if="resolvedLang" class="question-lang">{{ resolvedLang }}</span>
                </div>
            </div>
            <div ref="bodyRef" class="question-body">
                <slot />
            </div>
        </div>
    </div>
</template>

<style scoped>
.question-wrapper-parent {
    background: var(--c-bg);
    border-radius: 0.5rem;
}

.question-wrapper {
    border: 1px solid var(--c-fg);
    overflow: hidden;
}

.question-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0.5rem 1rem;
    border-bottom: 1px solid var(--c-fg);
    background: var(--c-bg);
}

.question-dot {
    width: 0.625rem;
    height: 0.625rem;
    border: 1.5px solid var(--c-fg);
}

.question-lang {
    font-family: var(--font-mono);
    font-size: 0.75rem;
    font-weight: 500;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    color: var(--c-muted-fg);
}

.question-body {
    padding: 1rem;
}

.question-body :deep(h3:first-child) {
    margin-top: 0;
}

.question-body :deep(.contains-task-list) {
    background: var(--c-muted);
    padding: 1rem 1.25rem;
    border-radius: 0.375rem;
}

.question-body :deep(li.quiz-option) {
    list-style: none;
    padding-left: 1.75rem;
    cursor: pointer;
    transition: color 0.15s ease;
}

.question-body :deep(li.quiz-option)::before {
    display: none;
}

.question-body :deep(li.quiz-option input[type="radio"]) {
    position: absolute;
    left: 0;
    top: 0.3em;
    width: 1.1rem;
    height: 1.1rem;
    margin: 0;
    appearance: none;
    -webkit-appearance: none;
    border: 1.5px solid var(--c-fg);
    border-radius: 50%;
    background: var(--c-bg);
    cursor: pointer;
    transition: border-color 0.15s ease, background-color 0.15s ease;
}

.question-body :deep(li.quiz-option input[type="radio"]:checked) {
    background: var(--c-fg);
    box-shadow: inset 0 0 0 3px var(--c-bg);
}

.question-body :deep(li.quiz-option.is-correct) {
    color: var(--c-success);
}

.question-body :deep(li.quiz-option.is-correct input[type="radio"]) {
    border-color: var(--c-success);
    background: var(--c-success);
    box-shadow: inset 0 0 0 3px var(--c-bg);
}

.question-body :deep(li.quiz-option.is-incorrect) {
    color: var(--c-error);
}

.question-body :deep(li.quiz-option.is-incorrect input[type="radio"]) {
    border-color: var(--c-error);
    background: var(--c-error);
    box-shadow: inset 0 0 0 3px var(--c-bg);
}
</style>
