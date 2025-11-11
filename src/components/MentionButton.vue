<template>
    <button
        type="button"
        class="ww-chat-input-area__mention-btn"
        :class="{ 'ww-chat-input-area__mention-btn--disabled': isUiDisabled }"
        :style="mentionButtonStyle"
        :disabled="isUiDisabled"
        @click="handleClick"
        aria-label="Add mention"
    >
        <span
            class="ww-chat-input-area__icon"
            :style="{ width: mentionIconSize, height: mentionIconSize }"
            v-html="mentionIconHtml"
        ></span>
    </button>
</template>

<script>
import { ref, computed, watchEffect, inject } from 'vue';

export default {
    name: 'MentionButton',
    props: {
        isDisabled: {
            type: Boolean,
            default: false,
        },
        mentionIcon: {
            type: String,
            default: 'at-sign',
        },
        mentionIconColor: {
            type: String,
            default: '#334155',
        },
        mentionIconSize: {
            type: String,
            default: '20px',
        },
        mentionButtonBgColor: {
            type: String,
            default: '#f8fafc',
        },
        mentionButtonHoverBgColor: {
            type: String,
            default: '#f1f5f9',
        },
        mentionButtonBorder: {
            type: String,
            default: '1px solid #e2e8f0',
        },
        mentionButtonBorderRadius: {
            type: String,
            default: '12px',
        },
        mentionButtonSize: {
            type: String,
            default: '42px',
        },
        mentionButtonBoxShadow: {
            type: String,
            default: '0 1px 2px rgba(0, 0, 0, 0.06)',
        },
    },
    emits: ['click'],
    setup(props, { emit }) {
        const isEditing = inject(
            'isEditing',
            computed(() => false)
        );

        const mentionIconText = ref(null);
        const { getIcon } = wwLib.useIcons();

        const defaultMentionIcon = `<svg
            xmlns="http://www.w3.org/2000/svg"
            width="20"
            height="20"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
        >
            <circle cx="12" cy="12" r="4"></circle>
            <path d="M16 8v5a3 3 0 0 0 6 0v-1a10 10 0 1 0-3.92 7.94"></path>
        </svg>`;

        watchEffect(async () => {
            try {
                if (props.mentionIcon) {
                    mentionIconText.value = await getIcon(props.mentionIcon);
                }
            } catch (error) {
                mentionIconText.value = null;
            }
        });

        const mentionIconHtml = computed(() => {
            return mentionIconText.value || defaultMentionIcon;
        });

        const isUiDisabled = computed(() => props.isDisabled || isEditing.value);

        const mentionButtonStyle = computed(() => ({
            color: props.mentionIconColor,
            background: props.mentionButtonBgColor,
            border: props.mentionButtonBorder,
            borderRadius: props.mentionButtonBorderRadius,
            width: props.mentionButtonSize,
            height: props.mentionButtonSize,
            boxShadow: props.mentionButtonBoxShadow,
            '--btn-hover-bg': props.mentionButtonHoverBgColor,
        }));

        const handleClick = () => {
            if (isEditing.value || props.isDisabled) return;
            emit('click');
        };

        return {
            isUiDisabled,
            mentionIconHtml,
            mentionButtonStyle,
            handleClick,
        };
    },
};
</script>

<style lang="scss" scoped>
.ww-chat-input-area__mention-btn {
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
    flex-shrink: 0;
    align-self: auto;

    &:hover:not(:disabled) {
        background: var(--btn-hover-bg, #f1f5f9);
        transform: translateY(-1px);
        box-shadow: var(--btn-hover-shadow, 0 2px 4px rgba(0, 0, 0, 0.1));
    }

    &--disabled {
        opacity: 0.5;
        cursor: not-allowed;
        pointer-events: none;
        box-shadow: none;
        transform: none;
    }

    &:active:not(:disabled) {
        transform: translateY(0);
        box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
    }
}

.ww-chat-input-area__icon {
    display: flex;
    align-items: center;
    justify-content: center;

    :deep(svg) {
        width: 100%;
        height: 100%;
    }
}
</style>
