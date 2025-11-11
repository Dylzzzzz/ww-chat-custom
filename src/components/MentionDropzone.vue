<template>
    <div
        class="ww-mention-dropzone"
        :class="{ 'ww-mention-dropzone--own': isOwnMessage }"
    >
        <!-- Render dropzone template content with mention data -->
        <div v-if="hasDropzoneContent" class="ww-mention-dropzone__custom">
            <wwLayoutItemContext
                :index="0"
                :item="{}"
                is-repeat
                :data="mentionData"
            >
                <wwLayout
                    path="mention-template"
                    :content="content"
                    :uid="uid"
                />
            </wwLayoutItemContext>
        </div>

        <!-- Default fallback rendering when dropzone is empty -->
        <div v-else class="ww-mention-dropzone__default">
            <span class="ww-mention-dropzone__label">@{{ mention.recordLabel }}</span>
        </div>
    </div>
</template>

<script>
import { computed, provide } from 'vue';

export default {
    name: 'MentionDropzone',
    props: {
        mention: {
            type: Object,
            required: true,
        },
        recordTypeId: {
            type: String,
            required: true,
        },
        isOwnMessage: {
            type: Boolean,
            default: false,
        },
        content: {
            type: Object,
            default: () => ({}),
        },
        uid: {
            type: String,
            default: '',
        },
    },
    setup(props) {
        // Get dropzone template content
        const dropzoneElements = computed(() => {
            return props.content?.['mention-template'] || [];
        });

        const hasDropzoneContent = computed(() => {
            return dropzoneElements.value && dropzoneElements.value.length > 0;
        });

        // Create mention context data
        const mentionData = computed(() => ({
            recordTypeId: props.mention.recordTypeId,
            recordId: props.mention.recordId,
            recordLabel: props.mention.recordLabel,
            recordData: props.mention.recordData || {},
            isOwnMessage: props.isOwnMessage,
        }));

        return {
            dropzoneElements,
            hasDropzoneContent,
            mentionData,
        };
    },
};
</script>

<style lang="scss" scoped>
.ww-mention-dropzone {
    margin-top: 8px;
    display: flex;
    flex-direction: column;
    gap: 4px;

    &--own {
        align-items: flex-end;
    }

    &__default {
        display: inline-flex;
        align-items: center;
        padding: 6px 12px;
        background-color: rgba(59, 130, 246, 0.1);
        border: 1px solid rgba(59, 130, 246, 0.3);
        border-radius: 8px;
        font-size: 0.875rem;
        font-weight: 500;
        color: #3b82f6;
        max-width: fit-content;
        transition: all 0.2s ease;

        &:hover {
            background-color: rgba(59, 130, 246, 0.15);
            border-color: rgba(59, 130, 246, 0.4);
        }
    }

    &__label {
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
    }

    &__custom {
        display: flex;
        flex-direction: column;
        gap: 4px;
    }

    // Styling for own messages
    &--own &__default {
        background-color: rgba(59, 130, 246, 0.15);
        border-color: rgba(59, 130, 246, 0.4);
    }
}
</style>
