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
    <div class="question-wrapper-parent">
        <div class="question-wrapper">
            <div class="question-header">
                <span v-if="resolvedLang" class="question-lang">
                    <span class="question-dot" />
                    {{ resolvedLang }}
                </span>
            </div>
            <div ref="bodyRef" class="question-body">
                <slot />
            </div>
        </div>
    </div>
</template>

<style scoped>
.question-wrapper-parent {
    margin-bottom: 2.5rem;
}

.question-wrapper {
    border: 1px solid var(--c-border-light);
    border-radius: 1rem;
    overflow: hidden;
    background: var(--c-muted);
    transition: border-color 0.15s ease;
}

.question-wrapper:hover {
    border-color: rgba(255, 255, 255, 0.2);
}

.question-header {
    display: flex;
    align-items: center;
    justify-content: flex-end;
    padding: 0 1.5rem;
    min-height: 0.5rem;
}

.question-header:has(.question-lang) {
    padding-top: 1.5rem;
}

.question-lang {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    border: 1px solid var(--c-border-light);
    background: rgba(255, 255, 255, 0.05);
    border-radius: 0.5rem;
    padding: 0.25rem 0.75rem;
    font-family: var(--font-mono);
    font-size: 0.75rem;
    color: var(--c-muted-fg);
}

.question-dot {
    width: 0.375rem;
    height: 0.375rem;
    border-radius: 999px;
    background: #4ade80;
}

.question-body {
    padding: 1.5rem;
}

@media (min-width: 640px) {
    .question-body {
        padding: 2rem;
    }
}

.question-body :deep(h3:first-child) {
    margin-top: 0;
    font-family: var(--font-mono);
    font-size: 1.125rem;
    font-weight: 700;
}

.question-body :deep(.contains-task-list) {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
    background: transparent;
    padding: 0;
    border-radius: 0;
    margin-bottom: 0;
}

.question-body :deep(li.quiz-option) {
    list-style: none;
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 0.75rem 1rem;
    border: 1px solid var(--c-border-light);
    background: rgba(255, 255, 255, 0.02);
    border-radius: 0.75rem;
    cursor: pointer;
    font-family: var(--font-mono);
    font-size: 0.875rem;
    transition: border-color 0.15s ease, background-color 0.15s ease;
}

.question-body :deep(li.quiz-option):hover {
    border-color: rgba(255, 255, 255, 0.25);
    background-color: rgba(255, 255, 255, 0.05);
}

.question-body :deep(li.quiz-option)::before {
    display: none;
}

.question-body :deep(li.quiz-option input[type="radio"]) {
    position: static;
    width: 1.1rem;
    height: 1.1rem;
    margin: 0;
    flex-shrink: 0;
    appearance: none;
    -webkit-appearance: none;
    border: 1.5px solid var(--c-border-light);
    border-radius: 50%;
    background: var(--c-bg);
    cursor: pointer;
    transition: border-color 0.15s ease, background-color 0.15s ease;
}

.question-body :deep(li.quiz-option input[type="radio"]:checked) {
    background: var(--c-fg);
    box-shadow: inset 0 0 0 3px var(--c-muted);
}

.question-body :deep(li.quiz-option.is-correct) {
    color: var(--c-success);
    border-color: var(--c-success);
}

.question-body :deep(li.quiz-option.is-correct input[type="radio"]) {
    border-color: var(--c-success);
    background: var(--c-success);
    box-shadow: inset 0 0 0 3px var(--c-muted);
}

.question-body :deep(li.quiz-option.is-incorrect) {
    color: var(--c-error);
    border-color: var(--c-error);
}

.question-body :deep(li.quiz-option.is-incorrect input[type="radio"]) {
    border-color: var(--c-error);
    background: var(--c-error);
    box-shadow: inset 0 0 0 3px var(--c-muted);
}

.question-body :deep(details) {
    margin-top: 1.25rem;
    border: 1px solid var(--c-border-light);
    background: rgba(255, 255, 255, 0.02);
    border-radius: 0.75rem;
    overflow: hidden;
}

.question-body :deep(details summary) {
    list-style: none;
    cursor: pointer;
    padding: 0.75rem 1rem;
    font-family: var(--font-mono);
    font-size: 0.75rem;
    color: var(--c-muted-fg);
    transition: color 0.15s ease;
}

.question-body :deep(details summary:hover) {
    color: var(--c-fg);
}

.question-body :deep(details summary::-webkit-details-marker) {
    display: none;
}

.question-body :deep(details[open] summary) {
    border-bottom: 1px solid var(--c-border-light);
}

.question-body :deep(details > *:not(summary)) {
    padding: 0 1.25rem;
}

.question-body :deep(details > *:last-child) {
    padding-bottom: 1.25rem;
}

.question-body :deep(details > p:first-of-type) {
    padding-top: 1.25rem;
}
</style>
