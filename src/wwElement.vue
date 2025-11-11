<template>
    <div ref="chatRoot" class="ww-chat" :class="{ 'ww-chat--disabled': isDisabled }" :style="containerStyles">
        <!-- Mention Template Dropzone -->
        <div class="ww-chat__mention-template-editor" :class="{ 'ww-chat__mention-template-editor--hidden': !isEditing }">
            <div class="ww-chat__mention-template-header">
                <span class="ww-chat__mention-template-title">📝 Mention Template</span>
                <span class="ww-chat__mention-template-description">Design how @mentions appear in messages</span>
            </div>
            <wwLayout
                path="mention-template"
                class="ww-chat__mention-dropzone"
                :content="content"
                :uid="uid"
                :data="mentionTemplateData"
            />
        </div>
        
        <!-- Chat Header -->
        <ChatHeader
            v-if="displayHeader"
            :user-name="headerUserName"
            :user-avatar="headerUserAvatar"
            :user-location="headerUserLocation"
            :user-status="headerUserStatus"
            :avatar-bg-color="headerAvatarBgColor"
            :header-bg-color="headerBgColor"
            :text-color="headerTextColor"
            :header-border="headerBorder"
            :header-padding="headerPadding"
            :name-font-size="headerNameFontSize"
            :name-font-weight="headerNameFontWeight"
            :location-font-size="headerLocationFontSize"
            :location-opacity="headerLocationOpacity"
            :close-button-color="headerCloseButtonColor"
            :close-button-bg-hover="headerCloseButtonBgHover"
            :show-close-button="headerShowCloseButton"
            :participants="headerParticipants"
            @close="handleClose"
        />

        <!-- Messages Area -->
        <div ref="messagesContainer" class="ww-chat__messages" :style="messagesContainerStyles">
            <MessageList
                :messages="messages"
                :current-user-id="currentUserId"
                :message-bg-color="messageBgColor"
                :message-text-color="messageTextColor"
                :message-font-size="messageFontSize"
                :message-font-weight="messageFontWeight"
                :message-font-family="messageFontFamily"
                :message-border="messageBorder"
                :message-radius="messageRadius"
                :own-message-bg-color="ownMessageBgColor"
                :own-message-text-color="ownMessageTextColor"
                :own-message-font-size="ownMessageFontSize"
                :own-message-font-weight="ownMessageFontWeight"
                :own-message-font-family="ownMessageFontFamily"
                :own-message-border="ownMessageBorder"
                :own-message-radius="ownMessageRadius"
                :empty-message-text="emptyMessageText"
                :empty-message-color="emptyMessageColor"
                :date-separator-text-color="dateSeparatorTextColor"
                :date-separator-line-color="dateSeparatorLineColor"
                :date-separator-bg-color="dateSeparatorBgColor"
                :date-separator-border-radius="dateSeparatorBorderRadius"
                :content="content"
                :uid="uid"
                @attachment-click="handleAttachmentClick"
                @message-right-click="handleMessageRightClick"
            />
        </div>

        <!-- Input Area -->
        <InputArea
            v-model="newMessage"
            :is-disabled="isDisabled"
            :allow-attachments="allowAttachments"
            :pending-attachments="pendingAttachments"
            :mention-enabled="mentionEnabled"
            :mention-record-types="processedMentionRecordTypes"
            :mention-icon="mentionIcon"
            :mention-icon-color="mentionIconColor"
            :mention-icon-size="mentionIconSize"
            :mention-button-bg-color="mentionButtonBgColor"
            :mention-button-hover-bg-color="mentionButtonHoverBgColor"
            :mention-button-border="mentionButtonBorder"
            :mention-button-border-radius="mentionButtonBorderRadius"
            :mention-button-size="mentionButtonSize"
            :mention-button-box-shadow="mentionButtonBoxShadow"
            :mention-dropdown-bg-color="mentionDropdownBgColor"
            :mention-dropdown-border="mentionDropdownBorder"
            :mention-dropdown-border-radius="mentionDropdownBorderRadius"
            :mention-dropdown-box-shadow="mentionDropdownBoxShadow"
            :mention-dropdown-max-height="mentionDropdownMaxHeight"
            :mention-dropdown-max-width="mentionDropdownMaxWidth"
            :mention-tab-text-color="mentionTabTextColor"
            :mention-tab-active-color="mentionTabActiveColor"
            :mention-tab-hover-color="mentionTabHoverColor"
            :mention-record-bg-color="mentionRecordBgColor"
            :mention-record-hover-bg-color="mentionRecordHoverBgColor"
            :mention-record-text-color="mentionRecordTextColor"
            :mention-record-font-size="mentionRecordFontSize"
            :input-bg-color="inputBgColor"
            :input-text-color="inputTextColor"
            :input-font-size="inputFontSize"
            :input-font-weight="inputFontWeight"
            :input-font-family="inputFontFamily"
            :input-placeholder-color="inputPlaceholderColor"
            :input-area-border="inputAreaBorder"
            :textarea-border="textareaBorder"
            :textarea-border-hover="textareaBorderHover"
            :textarea-border-focus="textareaBorderFocus"
            :input-height="inputHeight"
            :input-border-radius="inputBorderRadius"
            :placeholder="inputPlaceholder"
            :action-align="actionAlign"
            :send-icon="sendIcon"
            :send-icon-color="sendIconColor"
            :send-icon-size="sendIconSize"
            :attachment-icon="attachmentIcon"
            :attachment-icon-color="attachmentIconColor"
            :attachment-icon-size="attachmentIconSize"
            :remove-icon="removeIcon"
            :remove-icon-color="removeIconColor"
            :remove-icon-size="removeIconSize"
            :send-button-bg-color="sendButtonBgColor"
            :send-button-hover-bg-color="sendButtonHoverBgColor"
            :send-button-border="sendButtonBorder"
            :send-button-border-radius="sendButtonBorderRadius"
            :send-button-size="sendButtonSize"
            :send-button-box-shadow="sendButtonBoxShadow"
            :attachment-button-bg-color="attachmentButtonBgColor"
            :attachment-button-hover-bg-color="attachmentButtonHoverBgColor"
            :attachment-button-border="attachmentButtonBorder"
            :attachment-button-border-radius="attachmentButtonBorderRadius"
            :attachment-button-size="attachmentButtonSize"
            :attachment-button-box-shadow="attachmentButtonBoxShadow"
            @send="sendMessage"
            @attachment="handleAttachment"
            @remove-attachment="handleRemoveAttachment"
            @pending-attachment-click="handlePendingAttachmentClick"
            @mention-selected="handleMentionSelected"
            @mention-dropdown-opened="handleMentionDropdownOpened"
            @mention-dropdown-closed="handleMentionDropdownClosed"
        />
    </div>
</template>

<script>
import { ref, computed, watch, nextTick, provide, onMounted } from 'vue';
import ChatHeader from './components/ChatHeader.vue';
import MessageList from './components/MessageList.vue';
import InputArea from './components/InputArea.vue';

import {
    enUS,
    enGB,
    enCA,
    enAU,
    enNZ,
    enIE,
    enIN,
    enZA,
    fr,
    frCA,
    frCH,
    de,
    deAT,
    es,
    it,
    itCH,
    pt,
    ptBR,
    ru,
    ja,
    jaHira,
    zhCN as zh,
    zhHK,
    zhTW,
    ko,
    ar,
    arDZ,
    arEG,
    arMA,
    arSA,
    arTN,
    hi,
    bn,
    nl,
    nlBE,
    sv,
    nb,
    nn,
    da,
    fi,
    el,
    tr,
    cs,
    pl,
    ro,
    hu,
    vi,
    th,
    id,
    ms,
    uk,
} from 'date-fns/locale';

export default {
    name: 'Chat',
    components: {
        ChatHeader,
        MessageList,
        InputArea,
    },
    props: {
        content: {
            type: Object,
            required: true,
        },
        /* wwEditor:start */
        wwEditorState: {
            type: Object,
            required: true,
        },
        /* wwEditor:end */
        uid: {
            type: String,
            required: true,
        },
        wwElementState: {
            type: Object,
            required: true,
        },
    },
    emits: ['trigger-event'],
    setup(props, { emit }) {
        const chatRoot = ref(null);
        const messagesContainer = ref(null);
        const newMessage = ref('');
        const isScrolling = ref(false);
        const pendingAttachments = ref([]);

        const debounce = (func, delay) => {
            let timeoutId;
            return (...args) => {
                clearTimeout(timeoutId);
                timeoutId = setTimeout(() => func.apply(null, args), delay);
            };
        };

        const { value: chatState, setValue: setChatState } = wwLib.wwVariable.useComponentVariable({
            uid: props.uid,
            name: 'chatState',
            type: 'object',
            defaultValue: {
                messages: [],
                conversation: {
                    type: 'private',
                    participantCount: 1,
                    otherParticipantCount: 0,
                    participants: [],
                    allParticipants: [],
                },
                currentUser: { id: '', name: '', avatar: '', location: '', status: 'online' },
                utils: { messageCount: 0, isDisabled: false, allowAttachments: false, displayHeader: true },
            },
        });

        const { resolveMappingFormula } = wwLib.wwFormula.useFormula();

        const resolveMapping = (message, mappingFormula, defaultProp) => {
            if (!message || typeof message !== 'object') return '';
            if (!mappingFormula) return message[defaultProp];
            return resolveMappingFormula(mappingFormula, message);
        };

        const isEditing = computed(() => {
            /* wwEditor:start */
            return props.wwEditorState.isEditing;
            /* wwEditor:end */
            // eslint-disable-next-line no-unreachable
            return false;
        });

        // Participants mapping
        const resolveParticipantMapping = (obj, formula, fallbackKey) => {
            if (!obj || typeof obj !== 'object') return undefined;
            if (!formula) return obj[fallbackKey];
            return resolveMappingFormula(formula, obj);
        };

        const rawParticipants = computed(() => props.content?.participants || []);

        const participants = computed(() => {
            if (!Array.isArray(rawParticipants.value)) return [];
            return rawParticipants.value
                .map(p => {
                    const id = resolveParticipantMapping(p, props.content?.mappingParticipantId, 'id');
                    const name =
                        resolveParticipantMapping(p, props.content?.mappingParticipantName, 'name') ||
                        resolveParticipantMapping(p, props.content?.mappingParticipantName, 'userName') ||
                        'User';
                    const avatar =
                        resolveParticipantMapping(p, props.content?.mappingParticipantAvatar, 'avatar') ||
                        resolveParticipantMapping(p, props.content?.mappingParticipantAvatar, 'avatarUrl') ||
                        '';
                    const location = resolveParticipantMapping(p, props.content?.mappingParticipantLocation, 'location') || '';
                    const status = resolveParticipantMapping(p, props.content?.mappingParticipantStatus, 'status') || 'online';
                    const isCurrentUser = !!resolveParticipantMapping(p, props.content?.mappingIsCurrentUser, 'isCurrentUser');
                    return { id, name, avatar, location, status, isCurrentUser };
                })
                .filter(p => !!p.id);
        });

        const currentUserId = computed(() => participants.value.find(p => p.isCurrentUser)?.id || '');
        const rawMessages = computed(() => {
            // Ensure we always have an array to work with
            const messagesContent = props.content?.messages;

            // Check if messages is a valid array
            if (Array.isArray(messagesContent)) {
                return messagesContent;
            }

            // Final fallback to empty array
            return [];
        });

        const messages = computed(() => {
            const mapAttachmentField = (obj, formula, fallbackKey) => {
                if (!obj || typeof obj !== 'object') return undefined;
                if (!formula) return obj[fallbackKey];
                return resolveMappingFormula(formula, obj);
            };

            return rawMessages.value.map(message => {
                if (!message || typeof message !== 'object') {
                    return {
                        id: `msg-${wwLib.wwUtils.getUid()}`,
                        text: '',
                        senderId: '',
                        userName: '',
                        avatar: '',
                        location: '',
                        status: '',
                        timestamp: '',
                        attachments: [],
                        userSettings: {},
                        _originalData: message,
                    };
                }

                const senderId = resolveMapping(message, props.content?.mappingSenderId, 'senderId') || '';
                const participant = participants.value.find(p => p.id === senderId);
                const displayUserName = participant?.name || 'Unknown User';
                const userSettings = {
                    userName: participant?.name || '',
                    userAvatar: participant?.avatar || '',
                    userLocation: participant?.location || '',
                    userStatus: participant?.status || 'online',
                };

                // Map attachments with optional field mappings
                const rawAttachments = resolveMapping(message, props.content?.mappingAttachments, 'attachments');
                let attachments;
                if (Array.isArray(rawAttachments)) {
                    attachments = rawAttachments.map(att => {
                        if (!att || typeof att !== 'object') return att;
                        return {
                            id: mapAttachmentField(att, props.content?.mappingAttachmentId, 'id'),
                            name: mapAttachmentField(att, props.content?.mappingAttachmentName, 'name'),
                            url: mapAttachmentField(att, props.content?.mappingAttachmentUrl, 'url'),
                            type: mapAttachmentField(att, props.content?.mappingAttachmentType, 'type'),
                            size: mapAttachmentField(att, props.content?.mappingAttachmentSize, 'size'),
                            // Preserve local upload File if present
                            file: att.file,
                        };
                    });
                } else {
                    attachments = rawAttachments;
                }

                // Map mentions with optional field mapping
                const rawMentions = resolveMapping(message, props.content?.mappingMessageMentions, 'mentions');
                let mentions;
                if (Array.isArray(rawMentions)) {
                    mentions = rawMentions;
                } else {
                    mentions = rawMentions;
                }

                return {
                    id:
                        resolveMapping(message, props.content?.mappingMessageId, 'id') ||
                        `msg-${wwLib.wwUtils.getUid()}`,
                    text: resolveMapping(message, props.content?.mappingMessageText, 'text') || '',
                    senderId: senderId,
                    userName: displayUserName,
                    avatar: userSettings.userAvatar,
                    location: userSettings.userLocation,
                    status: userSettings.userStatus,
                    timestamp:
                        resolveMapping(message, props.content?.mappingTimestamp, 'timestamp') ||
                        new Date().toISOString(),
                    attachments,
                    mentions,
                    userSettings: userSettings,
                    _originalData: message,
                };
            });
        });

        const isDisabled = computed(() => props.content?.disabled || false);
        const displayHeader = computed(() => props.content?.displayHeader !== false);
        const allowAttachments = computed(() => props.content?.allowAttachments || false);
        const inputPlaceholder = computed(() => props.content?.inputPlaceholder || 'Type a message...');

        // Mention system computed properties
        const mentionEnabled = computed(() => props.content?.mentionEnabled || false);

        const resolveRecordTypeMapping = (recordType, formula, fallbackKey) => {
            if (!recordType || typeof recordType !== 'object') return undefined;
            if (!formula) return recordType[fallbackKey];
            return resolveMappingFormula(formula, recordType);
        };

        const resolveRecordMapping = (record, formula, fallbackKey) => {
            if (!record || typeof record !== 'object') return undefined;
            if (!formula) return record[fallbackKey];
            return resolveMappingFormula(formula, record);
        };

        const processedMentionRecordTypes = computed(() => {
            if (!mentionEnabled.value || !props.content?.mentionRecordTypes) return [];

            const rawTypes = Array.isArray(props.content.mentionRecordTypes) ? props.content.mentionRecordTypes : [];

            return rawTypes
                .map(type => {
                    const id = resolveRecordTypeMapping(type, props.content?.mappingRecordTypeId, 'id');
                    const label = resolveRecordTypeMapping(type, props.content?.mappingRecordTypeLabel, 'label') ||
                                  resolveRecordTypeMapping(type, props.content?.mappingRecordTypeLabel, 'name');
                    const icon = resolveRecordTypeMapping(type, props.content?.mappingRecordTypeIcon, 'icon');
                    const rawRecords = resolveRecordTypeMapping(type, props.content?.mappingRecordTypeRecords, 'records') ||
                                       resolveRecordTypeMapping(type, props.content?.mappingRecordTypeRecords, 'items') ||
                                       [];

                    const records = Array.isArray(rawRecords)
                        ? rawRecords.map(record => ({
                              id: resolveRecordMapping(record, props.content?.mappingRecordId, 'id'),
                              label: resolveRecordMapping(record, props.content?.mappingRecordLabel, 'label') ||
                                     resolveRecordMapping(record, props.content?.mappingRecordLabel, 'name') ||
                                     resolveRecordMapping(record, props.content?.mappingRecordLabel, 'title'),
                              avatar: resolveRecordMapping(record, props.content?.mappingRecordAvatar, 'avatar') ||
                                      resolveRecordMapping(record, props.content?.mappingRecordAvatar, 'icon') ||
                                      resolveRecordMapping(record, props.content?.mappingRecordAvatar, 'image'),
                              metadata: resolveRecordMapping(record, props.content?.mappingRecordMetadata, 'metadata') ||
                                        resolveRecordMapping(record, props.content?.mappingRecordMetadata, 'data') ||
                                        record,
                          }))
                        : [];

                    return {
                        id,
                        label,
                        icon,
                        records,
                    };
                })
                .filter(type => type.id && type.label);
        });

        // User properties - use direct props since we no longer have global user settings
        const currentUserParticipant = computed(() => participants.value.find(p => p.isCurrentUser));

        // Style properties
        const containerStyles = computed(() => ({
            fontFamily: props.content?.fontFamily || 'inherit',
        }));

        const messagesAreaPadding = computed(() => props.content?.messagesAreaPadding || '16px');

        const messagesContainerStyles = computed(() => ({
            backgroundColor: props.content?.messagesAreaBgColor || '#ffffff',
            padding: messagesAreaPadding.value,
        }));

        // Header styles
        const headerBgColor = computed(() => props.content?.headerBgColor || '#ffffff');
        const headerTextColor = computed(() => props.content?.headerTextColor || '#1e293b');
        const headerBorder = computed(() => props.content?.headerBorder || '1px solid #e2e8f0');

        const headerPadding = computed(() => props.content?.headerPadding || '16px');
        const headerNameFontSize = computed(() => props.content?.headerNameFontSize || '18px');
        const headerNameFontWeight = computed(() => props.content?.headerNameFontWeight || '600');
        const headerLocationFontSize = computed(() => props.content?.headerLocationFontSize || '14px');
        const headerLocationOpacity = computed(() => props.content?.headerLocationOpacity || '0.7');
        const headerCloseButtonColor = computed(() => props.content?.headerCloseButtonColor || '#334155');
        const headerCloseButtonBgHover = computed(() => props.content?.headerCloseButtonBgHover || '#dbeafe');

        // Message styles
        const messageBgColor = computed(() => props.content?.messageBgColor || '#f1f5f9');
        const messageTextColor = computed(() => props.content?.messageTextColor || '#334155');
        const messageFontSize = computed(() => props.content?.messageFontSize || '0.875rem');
        const messageFontWeight = computed(() => props.content?.messageFontWeight || '400');
        const messageFontFamily = computed(() => props.content?.messageFontFamily || 'inherit');
        const messageBorder = computed(() => props.content?.messageBorder || '1px solid #e2e8f0');
        const messageRadius = computed(() => props.content?.messageRadius || '18px 18px 18px 18px');

        const ownMessageBgColor = computed(() => props.content?.ownMessageBgColor || '#dbeafe');
        const ownMessageTextColor = computed(() => props.content?.ownMessageTextColor || '#1e40af');
        const ownMessageFontSize = computed(() => props.content?.ownMessageFontSize || '0.875rem');
        const ownMessageFontWeight = computed(() => props.content?.ownMessageFontWeight || '400');
        const ownMessageFontFamily = computed(() => props.content?.ownMessageFontFamily || 'inherit');
        const ownMessageBorder = computed(() => props.content?.ownMessageBorder || '1px solid #bfdbfe');
        const ownMessageRadius = computed(() => props.content?.ownMessageRadius || '18px 18px 18px 18px');

        // Input styles
        const inputBgColor = computed(() => props.content?.inputBgColor || '#ffffff');
        const inputTextColor = computed(() => props.content?.inputTextColor || '#334155');
        const inputFontSize = computed(() => props.content?.inputFontSize || '0.875rem');
        const inputFontWeight = computed(() => props.content?.inputFontWeight || '400');
        const inputFontFamily = computed(() => props.content?.inputFontFamily || 'inherit');
        const inputPlaceholderColor = computed(() => props.content?.inputPlaceholderColor || '#94a3b8');
        const inputAreaBorder = computed(() => props.content?.inputAreaBorder || '1px solid #e2e8f0');
        const textareaBorder = computed(() => props.content?.textareaBorder || '1px solid #e2e8f0');
        const textareaBorderHover = computed(() => props.content?.textareaBorderHover || '1px solid #cbd5e1');
        const textareaBorderFocus = computed(() => props.content?.textareaBorderFocus || '1px solid #3b82f6');
        const inputHeight = computed(() => props.content?.inputHeight || '38px');
        const inputBorderRadius = computed(() => props.content?.inputBorderRadius || '8px');

        // Empty message styles
        const emptyMessageText = computed(() => props.content?.emptyMessageText || 'No messages yet');
        const emptyMessageColor = computed(() => props.content?.emptyMessageColor || '#64748b');

        // Date separator styles
        const dateSeparatorTextColor = computed(() => props.content?.dateSeparatorTextColor || '#64748b');
        const dateSeparatorLineColor = computed(() => props.content?.dateSeparatorLineColor || '#e2e8f0');
        const dateSeparatorBgColor = computed(() => props.content?.dateSeparatorBgColor || '#ffffff');
        const dateSeparatorBorderRadius = computed(() => props.content?.dateSeparatorBorderRadius || '8px');

        // Messages attachments thumbnail sizing (in messages area)
        const messagesAttachmentThumbMaxWidth = computed(() => props.content?.messagesAttachmentThumbMaxWidth || '250px');
        const messagesAttachmentThumbMaxHeight = computed(() => props.content?.messagesAttachmentThumbMaxHeight || '200px');
        const messagesAttachmentThumbBorderRadius = computed(
            () => props.content?.messagesAttachmentThumbBorderRadius || '6px'
        );

        watch(
            messages,
            () => {
                if (!isScrolling.value) scrollToBottom(); // Will use autoScrollBehavior setting
            },
            { deep: true }
        );

        // Removed user settings watcher and debounced updater; participants now drive user info

        const scrollToBottom = async (smooth = null) => {
            await nextTick();
            if (messagesContainer.value) {
                // Use the autoScrollBehavior setting if smooth parameter is not explicitly provided
                const behavior =
                    smooth !== null ? (smooth ? 'smooth' : 'auto') : props.content?.autoScrollBehavior || 'auto';

                messagesContainer.value.scrollTo({
                    top: messagesContainer.value.scrollHeight,
                    behavior: behavior,
                });
            }
        };

        const sendMessage = (data) => {
            if (isEditing.value || isDisabled.value || (!newMessage.value.trim() && pendingAttachments.value.length === 0)) return;

            const attachments = [...pendingAttachments.value];
            // For the emitted event, only expose File objects (no id/url metadata)
            const attachmentsForEvent = attachments
                .map(att => att && att.file)
                .filter(file => !!file);
            // Clear local pending attachments right after snapshotting
            pendingAttachments.value = [];

            const message = {
                id: `msg-${wwLib.wwUtils.getUid()}`,
                text: newMessage.value.trim(),
                senderId: currentUserId.value,
                userName: currentUserParticipant.value?.name || 'You',
                timestamp: new Date().toISOString(),
                // Emit attachments as File[] only, without id/url/name duplication
                attachments: attachmentsForEvent.length > 0 ? attachmentsForEvent : undefined,
                // Include mentions if provided
                mentions: data?.mentions && data.mentions.length > 0 ? data.mentions : undefined,
                userSettings: currentUserParticipant.value
                    ? {
                          userName: currentUserParticipant.value.name,
                          userAvatar: currentUserParticipant.value.avatar,
                          userLocation: currentUserParticipant.value.location,
                          userStatus: currentUserParticipant.value.status,
                      }
                    : {},
            };

            newMessage.value = '';

            emit('trigger-event', {
                name: 'messageSent',
                event: { message },
            });
        };

        const handleAttachment = files => {
            if (isEditing.value || isDisabled.value) return;

            const attachmentFiles = Array.from(files).map(file => ({
                id: `file-${wwLib.wwUtils.getUid()}`,
                name: file.name,
                type: file.type,
                size: file.size,
                url: URL.createObjectURL(file),
                file,
            }));

            pendingAttachments.value = [...pendingAttachments.value, ...attachmentFiles];
        };

        const handleRemoveAttachment = index => {
            if (isEditing.value || isDisabled.value) return;

            // Release the object URL to avoid memory leaks
            if (pendingAttachments.value[index]?.url) {
                URL.revokeObjectURL(pendingAttachments.value[index].url);
            }

            pendingAttachments.value.splice(index, 1);
        };

        const handleAttachmentClick = attachment => {
            if (isEditing.value) return;

            emit('trigger-event', {
                name: 'attachmentClick',
                event: { attachment },
            });
        };

        const handlePendingAttachmentClick = ({ attachment, index }) => {
            const file = attachment && attachment.file ? attachment.file : attachment;
            emit('trigger-event', {
                name: 'pendingAttachmentClick',
                event: { attachment: file, index },
            });
        };

        const handleMessageRightClick = ({ message, position }) => {
            if (isEditing.value) return;

            emit('trigger-event', {
                name: 'messageRightClick',
                event: { message, position },
            });
        };

        const handleClose = () => {
            if (isEditing.value) return;

            emit('trigger-event', {
                name: 'close',
                event: {},
            });
        };

        const handleMentionSelected = (data) => {
            if (isEditing.value) return;

            emit('trigger-event', {
                name: 'mentionSelected',
                event: data,
            });
        };

        const handleMentionDropdownOpened = () => {
            if (isEditing.value) return;

            emit('trigger-event', {
                name: 'mentionDropdownOpened',
                event: {},
            });
        };

        const handleMentionDropdownClosed = () => {
            if (isEditing.value) return;

            emit('trigger-event', {
                name: 'mentionDropdownClosed',
                event: {},
            });
        };

        const addMessage = message => {
            if (isEditing.value) return;

            const newMessageRaw = {
                id: message.id || `msg-${wwLib.wwUtils.getUid()}`,
                text: message.text || '',
                senderId: message.senderId || '',
                userName: message.userName || '',
                timestamp: message.timestamp || new Date().toISOString(),
                attachments: message.attachments,
                userSettings: message.userSettings || {
                    userName: message.userName || '',
                    userAvatar: message.userAvatar || '',
                    userLocation: message.userLocation || '',
                    userStatus: message.userStatus || 'online',
                },
                ...message,
            };

            const updatedMessages = [...(chatState.value?.messages || []), newMessageRaw];
            setChatState({
                ...chatState.value,
                messages: updatedMessages,
            });

            scrollToBottom();

            return newMessageRaw;
        };

        // Emit messageReceived for new external messages (not initial history, not own messages)
        let _messagesWatcherInitialized = false;
        const _seenMessageIds = new Set();
        watch(
            () => messages.value,
            newMessages => {
                if (isEditing.value) return;

                if (!_messagesWatcherInitialized) {
                    newMessages.forEach(m => {
                        if (m && m.id) _seenMessageIds.add(m.id);
                    });
                    _messagesWatcherInitialized = true;
                    return;
                }

                for (const m of newMessages) {
                    const id = m?.id;
                    if (!id) continue;
                    if (_seenMessageIds.has(id)) continue;
                    _seenMessageIds.add(id);
                    if (m.senderId && m.senderId !== currentUserId.value) {
                        emit('trigger-event', {
                            name: 'messageReceived',
                            event: { message: m },
                        });
                    }
                }
            },
            { deep: false }
        );

        // Date/time locale configuration
        const locale = computed(() => {
            if (!props.content?.locale) return enUS;

            const locales = {
                // English variants
                enUS,
                enGB,
                enCA,
                enAU,
                enNZ,
                enIE,
                enIN,
                enZA,
                // French variants
                fr,
                frCA,
                frCH,
                // German variants
                de,
                deAT,
                // Spanish
                es,
                // Italian variants
                it,
                itCH,
                // Portuguese variants
                pt,
                ptBR,
                // Russian
                ru,
                // East Asian languages
                ja,
                jaHira,
                zh,
                zhHK,
                zhTW,
                ko,
                // Arabic variants
                ar,
                arDZ,
                arEG,
                arMA,
                arSA,
                arTN,
                // Indian subcontinent languages
                hi,
                bn,
                // Other European
                nl,
                nlBE,
                sv,
                nb,
                nn,
                da,
                fi,
                el,
                tr,
                cs,
                pl,
                ro,
                hu,
                // Southeast Asian
                vi,
                th,
                id,
                ms,
                // Other
                uk,
            };

            return locales[props.content.locale] || enUS;
        });

        const dateTimeOptions = computed(() => ({
            locale: locale.value,
            timeFormat: props.content?.timeFormat,
            todayText: props.content?.todayText,
            yesterdayText: props.content?.yesterdayText,
            justNowText: props.content?.justNowText,
        }));

        provide('dateTimeOptions', dateTimeOptions);
        provide('chatRootEl', chatRoot);

        const chatPartners = computed(() => {
            // If no messages, return current user info as fallback
            if (participants.value.length === 0) {
                return { name: '', avatar: '', location: '', status: 'online', participants: [], participantsString: '', isGroup: false };
            }

            if (messages.value.length === 0) {
                return {
                    name: currentUserParticipant.value?.name || '',
                    avatar: currentUserParticipant.value?.avatar || '',
                    location: currentUserParticipant.value?.location || '',
                    status: currentUserParticipant.value?.status || 'online',
                    participants: [],
                    participantsString: '',
                    isGroup: false,
                };
            }

            // Get all unique sender IDs from messages
            const allSenderIds = [...new Set(messages.value.map(msg => msg.senderId))];
            const otherSenderIds = allSenderIds.filter(id => id !== currentUserId.value);

            const others = participants.value.filter(p => otherSenderIds.includes(p.id));
            const names = others.map(p => p.name);
            const participantsString = names.join(', ');

            // If no other participants (only current user messages), show current user
            if (otherSenderIds.length === 0) {
                return {
                    name: currentUserParticipant.value?.name || '',
                    avatar: currentUserParticipant.value?.avatar || '',
                    location: currentUserParticipant.value?.location || '',
                    status: currentUserParticipant.value?.status || 'online',
                    participants: [],
                    participantsString: '',
                };
            }

            // Two participants total (current user + 1 other) - show recipient info
            if (allSenderIds.length === 2 || (otherSenderIds.length === 1 && !props.content?.showSelfInHeader)) {
                const otherUserId = otherSenderIds[0];
                const p = participants.value.find(p => p.id === otherUserId);
                return {
                    name: p?.name || 'Unknown User',
                    avatar: p?.avatar || '',
                    location: p?.location || '',
                    status: p?.status || 'online',
                    participants: names,
                    participantsString,
                    isGroup: false,
                };
            }

            // Multiple participants (3+) - show group chat info
            const lastOtherMsg = [...messages.value]
                .filter(msg => msg.senderId !== currentUserId.value)
                .sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp))[0];

            // Use groupChatText if defined, otherwise use default template
            let groupChatName;
            if (props.content?.groupChatText && props.content.groupChatText.trim() !== '') {
                groupChatName = props.content.groupChatText;
            } else {
                const totalParticipants = others.length + (currentUserParticipant.value ? 1 : 0);
                groupChatName = `Group Chat (${totalParticipants} participants)`;
            }

            // For group chat location, use userSettings from the last message sender
            let groupLocation = '';
            if (lastOtherMsg) {
                const displayName = lastOtherMsg.userName;
                groupLocation = `Last message from ${displayName}`;
            }

            return {
                name: groupChatName,
                avatar: props.content?.groupChatAvatar || '',
                location: groupLocation,
                status: 'online',
                participants,
                participantsString,
                isGroup: true,
            };
        });

        const headerUserName = computed(() => chatPartners.value?.name || '');
        const headerUserAvatar = computed(() => chatPartners.value?.avatar || '');
        const headerUserLocation = computed(() => chatPartners.value?.location || '');
        const headerUserStatus = computed(() => chatPartners.value?.status || 'online');
        const headerParticipants = computed(() => chatPartners.value?.participantsString || '');
        const headerAvatarBgColor = computed(() => {
            if (chatPartners.value?.isGroup) {
                return props.content?.groupChatAvatarColor || '';
            }
            return '';
        });

        // Local context functionality
        const currentLocalContext = ref({});

        const registerChatLocalContext = ({ data, markdown }) => {
            wwLib.wwElement.useRegisterElementLocalContext('chat', data, {}, markdown);
            currentLocalContext.value = { data, markdown };
        };

        // Chat local context data
        const conversationData = computed(() => {
            const others = participants.value.filter(p => p.id !== currentUserId.value);
            const all = participants.value.map(p => ({ ...p, isCurrentUser: p.id === currentUserId.value }));

            return {
                type: all.length <= 2 ? 'private' : 'group',
                participantCount: all.length,
                otherParticipantCount: others.length,
                participants: others.map(p => ({
                    id: p.id,
                    name: p.name,
                    avatar: p.avatar || '',
                    location: p.location || '',
                    status: p.status || 'online',
                    lastMessageTime: messages.value
                        .filter(m => m.senderId === p.id)
                        .sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp))[0]?.timestamp,
                })),
                allParticipants: all.map(p => ({
                    id: p.id,
                    name: p.name,
                    avatar: p.avatar || '',
                    location: p.location || '',
                    status: p.status || 'online',
                    isCurrentUser: p.isCurrentUser,
                    lastMessageTime: messages.value
                        .filter(m => m.senderId === p.id)
                        .sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp))[0]?.timestamp,
                })),
            };
        });

        const chatData = computed(() => ({
            messages: messages.value.map((message, index) => ({
                ...message,
                isOwn: message.senderId === currentUserId.value,
                isFirst: index === 0 || messages.value[index - 1].senderId !== message.senderId,
                isLast: index === messages.value.length - 1 || messages.value[index + 1].senderId !== message.senderId,
                participantInfo: conversationData.value.allParticipants.find(p => p.id === message.senderId) || {
                    id: message.senderId,
                    name: message.userSettings?.userName || message.userName || 'Unknown User',
                    avatar: message.userSettings?.userAvatar || message.avatar || '',
                    location: message.userSettings?.userLocation || message.location || '',
                    status: message.userSettings?.userStatus || message.status || 'online',
                    isCurrentUser: message.senderId === currentUserId.value,
                },
            })),
            conversation: conversationData.value,
            currentUser: {
                id: currentUserId.value,
                name: currentUserParticipant.value?.name || '',
                avatar: currentUserParticipant.value?.avatar || '',
                location: currentUserParticipant.value?.location || '',
                status: currentUserParticipant.value?.status || 'online',
            },
            utils: {
                messageCount: messages.value.length,
                isDisabled: isDisabled.value,
                allowAttachments: allowAttachments.value,
                displayHeader: displayHeader.value,
            },
        }));

        const chatMarkdown = `### Chat local information

        #### messages
        Array of all messages in the conversation. Each message contains:
        - \`id\`: Unique message identifier
        - \`text\`: Message content
        - \`senderId\`: ID of the message sender
        - \`userName\`: Name of the message sender
        - \`timestamp\`: Message timestamp (ISO string)
        - \`attachments\`: Message attachments (if any)
        - \`isOwn\`: Boolean indicating if message is from current user
        - \`isFirst\`: Boolean indicating if this is first message in a group from this sender
        - \`isLast\`: Boolean indicating if this is last message in a group from this sender
        - \`participantInfo\`: Information about the sender (id, name, avatar, isCurrentUser)

        #### conversation
        Information about the conversation:
        - \`type\`: Conversation type ('private' for 2 participants, 'group' for 3+)
        - \`participantCount\`: Total number of participants including current user
        - \`otherParticipantCount\`: Number of other participants (excluding current user)
        - \`participants\`: Array of other participants (excluding current user)
        - \`allParticipants\`: Array of all participants including current user

        #### currentUser
        Information about the current user:
        - \`id\`: Current user ID
        - \`name\`: Current user name
        - \`avatar\`: Current user avatar URL
        - \`location\`: Current user location
        - \`status\`: Current user status

        #### userSettings (per message)
        Each message contains a userSettings object with user information:
        - \`userName\`: User's display name
        - \`userAvatar\`: User's avatar URL
        - \`userLocation\`: User's location
        - \`userStatus\`: User's status (online, offline, away, busy)
        - Automatically updated for current user's messages when settings change

        #### utils
        Component state information:
        - \`messageCount\`: Total number of messages
        - \`isDisabled\`: Boolean indicating if chat is disabled
        - \`allowAttachments\`: Boolean indicating if attachments are allowed
        - \`displayHeader\`: Boolean indicating if header is displayed`;

        // Sync chatState with local context data
        watch(
            chatData,
            newChatData => {
                setChatState({
                    ...newChatData,
                });
            },
            { deep: true, immediate: true }
        );

        registerChatLocalContext({
            data: chatData,
            markdown: chatMarkdown,
        });

        // Register mention template context for dropzone
        const mentionTemplateData = computed(() => ({
            // Individual mention data (what each mention card receives)
            mention: {
                recordTypeId: 'users',
                recordId: '1',
                recordLabel: 'John Doe',
                recordData: {
                    email: 'john@example.com',
                    role: 'Developer',
                    avatar: 'https://i.pravatar.cc/150?img=1',
                    status: 'online',
                },
            },
            // Message context (parent message containing this mention)
            message: {
                id: 'msg-1',
                text: 'Hey @John Doe, can you review this?',
                senderId: 'user-2',
                userName: 'Jane Smith',
                timestamp: new Date().toISOString(),
                mentions: [
                    {
                        recordTypeId: 'users',
                        recordId: '1',
                        recordLabel: 'John Doe',
                        recordData: { email: 'john@example.com', role: 'Developer' },
                    },
                ],
            },
            isOwnMessage: false,
        }));

        const mentionTemplateMarkdown = `### Mention Template Context

Available data for designing mention cards:

#### mention
The individual mention object:
- \`mention.recordTypeId\`: Type of record (e.g., "users", "tasks", "projects")
- \`mention.recordId\`: Unique identifier for the mentioned record
- \`mention.recordLabel\`: Display name/label for the mentioned record
- \`mention.recordData\`: Additional metadata (object with custom fields like email, role, avatar, etc.)

#### message
The parent message containing this mention:
- \`message.id\`: Message ID
- \`message.text\`: Full message text
- \`message.senderId\`: ID of message sender
- \`message.userName\`: Name of message sender
- \`message.timestamp\`: Message timestamp
- \`message.mentions\`: Array of all mentions in this message

#### isOwnMessage
Boolean indicating if the mention is in the current user's message`;

        wwLib.wwElement.useRegisterElementLocalContext(
            'mention-template',
            mentionTemplateData,
            {},
            mentionTemplateMarkdown
        );

        provide('isEditing', isEditing);
        provide('_wwChat:localContext', currentLocalContext);

        onMounted(() => {
            // Ensure we show latest messages on mount
            scrollToBottom();
        });

        return {
            chatRoot,
            messagesContainer,
            newMessage,
            messages,
            pendingAttachments,

            currentUserId,
            participants,
            isDisabled,
            displayHeader,
            allowAttachments,
            inputPlaceholder,
            

            headerUserName,
            headerUserAvatar,
            headerUserLocation,
            headerUserStatus,
            headerParticipants,

            containerStyles,
            messagesContainerStyles,
            headerBgColor,
            headerTextColor,
            headerBorder,

            headerPadding,
            headerNameFontSize,
            headerNameFontWeight,
            headerLocationFontSize,
            headerLocationOpacity,
            headerCloseButtonColor,
            headerCloseButtonBgHover,
            headerShowCloseButton: computed(() => props.content?.headerShowCloseButton !== false),
            messageBgColor,
            messageTextColor,
            messageFontSize,
            messageFontWeight,
            messageFontFamily,
            messageBorder,
            messageRadius,
            messagesAreaPadding,
            ownMessageBgColor,
            ownMessageTextColor,
            ownMessageFontSize,
            ownMessageFontWeight,
            ownMessageFontFamily,
            ownMessageBorder,
            ownMessageRadius,
            inputBgColor,
            inputTextColor,
            inputFontSize,
            inputFontWeight,
            inputFontFamily,
            inputPlaceholderColor,
            inputAreaBorder,
            textareaBorder,
            textareaBorderHover,
            textareaBorderFocus,
            inputHeight,
            inputBorderRadius,

            emptyMessageText,
            emptyMessageColor,

            dateSeparatorTextColor,
            dateSeparatorLineColor,
            dateSeparatorBgColor,
            dateSeparatorBorderRadius,

            // Icons
            sendIcon: computed(() => props.content?.sendIcon || 'send'),
            sendIconColor: computed(() => props.content?.sendIconColor || '#334155'),
            sendIconSize: computed(() => props.content?.sendIconSize || '20px'),
            attachmentIcon: computed(() => props.content?.attachmentIcon || 'paperclip'),
            attachmentIconColor: computed(() => props.content?.attachmentIconColor || '#334155'),
            attachmentIconSize: computed(() => props.content?.attachmentIconSize || '20px'),
            removeIcon: computed(() => props.content?.removeIcon || 'x'),
            removeIconColor: computed(() => props.content?.removeIconColor || '#f43f5e'),
            removeIconSize: computed(() => props.content?.removeIconSize || '12px'),

            // Mention system
            mentionEnabled,
            processedMentionRecordTypes,
            mentionIcon: computed(() => props.content?.mentionIcon || 'at-sign'),
            mentionIconColor: computed(() => props.content?.mentionIconColor || '#334155'),
            mentionIconSize: computed(() => props.content?.mentionIconSize || '20px'),
            mentionButtonBgColor: computed(() => props.content?.mentionButtonBgColor || '#f8fafc'),
            mentionButtonHoverBgColor: computed(() => props.content?.mentionButtonHoverBgColor || '#f1f5f9'),
            mentionButtonBorder: computed(() => props.content?.mentionButtonBorder || '1px solid #e2e8f0'),
            mentionButtonBorderRadius: computed(() => props.content?.mentionButtonBorderRadius || '12px'),
            mentionButtonSize: computed(() => props.content?.mentionButtonSize || '42px'),
            mentionButtonBoxShadow: computed(() => props.content?.mentionButtonBoxShadow || '0 1px 2px rgba(0, 0, 0, 0.06)'),
            mentionDropdownBgColor: computed(() => props.content?.mentionDropdownBgColor || '#ffffff'),
            mentionDropdownBorder: computed(() => props.content?.mentionDropdownBorder || '1px solid #e2e8f0'),
            mentionDropdownBorderRadius: computed(() => props.content?.mentionDropdownBorderRadius || '12px'),
            mentionDropdownBoxShadow: computed(() => props.content?.mentionDropdownBoxShadow || '0 4px 12px rgba(0, 0, 0, 0.15)'),
            mentionDropdownMaxHeight: computed(() => props.content?.mentionDropdownMaxHeight || '300px'),
            mentionDropdownMaxWidth: computed(() => props.content?.mentionDropdownMaxWidth || '400px'),
            mentionTabTextColor: computed(() => props.content?.mentionTabTextColor || '#64748b'),
            mentionTabActiveColor: computed(() => props.content?.mentionTabActiveColor || '#3b82f6'),
            mentionTabHoverColor: computed(() => props.content?.mentionTabHoverColor || '#334155'),
            mentionRecordBgColor: computed(() => props.content?.mentionRecordBgColor || 'transparent'),
            mentionRecordHoverBgColor: computed(() => props.content?.mentionRecordHoverBgColor || '#f1f5f9'),
            mentionRecordTextColor: computed(() => props.content?.mentionRecordTextColor || '#334155'),
            mentionRecordFontSize: computed(() => props.content?.mentionRecordFontSize || '0.875rem'),

            // Input action alignment and button styles
            actionAlign: computed(() => props.content?.inputActionAlign || 'end'),
            sendButtonBgColor: computed(() => props.content?.sendButtonBgColor || 'linear-gradient(135deg, #3b82f6, #2563eb)'),
            sendButtonHoverBgColor: computed(() => props.content?.sendButtonHoverBgColor || 'linear-gradient(135deg, #2563eb, #1d4ed8)'),
            sendButtonBorder: computed(() => props.content?.sendButtonBorder || 'none'),
            sendButtonBorderRadius: computed(() => props.content?.sendButtonBorderRadius || '12px'),
            sendButtonSize: computed(() => props.content?.sendButtonSize || '42px'),
            sendButtonBoxShadow: computed(() => props.content?.sendButtonBoxShadow || '0 2px 4px rgba(59, 130, 246, 0.3)'),
            attachmentButtonBgColor: computed(() => props.content?.attachmentButtonBgColor || '#f8fafc'),
            attachmentButtonHoverBgColor: computed(() => props.content?.attachmentButtonHoverBgColor || '#f1f5f9'),
            attachmentButtonBorder: computed(() => props.content?.attachmentButtonBorder || '1px solid #e2e8f0'),
            attachmentButtonBorderRadius: computed(() => props.content?.attachmentButtonBorderRadius || '12px'),
            attachmentButtonSize: computed(() => props.content?.attachmentButtonSize || '42px'),
            attachmentButtonBoxShadow: computed(() => props.content?.attachmentButtonBoxShadow || '0 1px 2px rgba(0, 0, 0, 0.06)'),

            // Methods
            scrollToBottom,
            sendMessage,
            handleAttachment,
            handleRemoveAttachment,
            handlePendingAttachmentClick,
            handleAttachmentClick,
            handleMessageRightClick,
            handleClose,
            handleMentionSelected,
            handleMentionDropdownOpened,
            handleMentionDropdownClosed,
            addMessage,
            currentLocalContext,
            // exposed for CSS variables
            messagesAttachmentThumbMaxWidth,
            messagesAttachmentThumbMaxHeight,
            messagesAttachmentThumbBorderRadius,
            headerAvatarBgColor,
        };
    },
    methods: {
        actionScrollToBottom(smooth = false) {
            this.scrollToBottom(smooth);
        },
    },
};
</script>

<style lang="scss" scoped>
.ww-chat {
    --ww-chat-font-family: v-bind('containerStyles.fontFamily');

    --ww-chat-header-bg: v-bind('headerBgColor');
    --ww-chat-header-text: v-bind('headerTextColor');
    --ww-chat-header-border: v-bind('headerBorder');

    --ww-chat-header-padding: v-bind('headerPadding');
    --ww-chat-header-name-font-size: v-bind('headerNameFontSize');
    --ww-chat-header-name-font-weight: v-bind('headerNameFontWeight');
    --ww-chat-header-location-font-size: v-bind('headerLocationFontSize');
    --ww-chat-header-location-opacity: v-bind('headerLocationOpacity');
    --ww-chat-header-close-button-color: v-bind('headerCloseButtonColor');
    --ww-chat-header-close-button-bg-hover: v-bind('headerCloseButtonBgHover');

    --ww-chat-messages-bg: v-bind('messagesContainerStyles.backgroundColor');

    --ww-chat-message-bg: v-bind('messageBgColor');
    --ww-chat-message-text: v-bind('messageTextColor');
    --ww-chat-message-font-size: v-bind('messageFontSize');
    --ww-chat-message-font-weight: v-bind('messageFontWeight');
    --ww-chat-message-font-family: v-bind('messageFontFamily');
    --ww-chat-message-border: v-bind('messageBorder');

    --ww-chat-own-message-bg: v-bind('ownMessageBgColor');
    --ww-chat-own-message-text: v-bind('ownMessageTextColor');
    --ww-chat-own-message-font-size: v-bind('ownMessageFontSize');
    --ww-chat-own-message-font-weight: v-bind('ownMessageFontWeight');
    --ww-chat-own-message-font-family: v-bind('ownMessageFontFamily');
    --ww-chat-own-message-border: v-bind('ownMessageBorder');

    --ww-chat-empty-message-text: v-bind('emptyMessageText');
    --ww-chat-empty-message-color: v-bind('emptyMessageColor');

    --ww-chat-date-separator-text-color: v-bind('dateSeparatorTextColor');
    --ww-chat-date-separator-line-color: v-bind('dateSeparatorLineColor');
    --ww-chat-date-separator-bg-color: v-bind('dateSeparatorBgColor');
    --ww-chat-date-separator-border-radius: v-bind('dateSeparatorBorderRadius');

    /* Attachment thumbnails in messages area */
    --ww-chat-attachment-thumb-max-width: v-bind('messagesAttachmentThumbMaxWidth');
    --ww-chat-attachment-thumb-max-height: v-bind('messagesAttachmentThumbMaxHeight');
    --ww-chat-attachment-thumb-radius: v-bind('messagesAttachmentThumbBorderRadius');

    --ww-chat-input-bg: v-bind('inputBgColor');
    --ww-chat-input-text: v-bind('inputTextColor');
    --ww-chat-input-font-size: v-bind('inputFontSize');
    --ww-chat-input-font-weight: v-bind('inputFontWeight');
    --ww-chat-input-font-family: v-bind('inputFontFamily');
    --ww-chat-input-placeholder: v-bind('inputPlaceholderColor');
    --ww-chat-input-area-border: v-bind('inputAreaBorder');
    --ww-chat-textarea-border: v-bind('textareaBorder');
    --ww-chat-textarea-border-hover: v-bind('textareaBorderHover');
    --ww-chat-textarea-border-focus: v-bind('textareaBorderFocus');
    --ww-chat-input-height: v-bind('inputHeight');
    --ww-chat-input-border-radius: v-bind('inputBorderRadius');

    --ww-chat-messages-padding: v-bind('messagesAreaPadding');

    display: flex;
    flex-direction: column;
    overflow: hidden;

    font-family: var(--ww-chat-font-family);

    &--disabled {
        opacity: 0.7;
        pointer-events: none;
    }

    .ww-chat-header {
        flex-shrink: 0;
        z-index: 2;
    }
    &__messages {
        flex: 1;
        min-height: 0;
        overflow-y: auto;
        padding: var(--ww-chat-messages-padding);
        background-color: var(--ww-chat-messages-bg);
        position: relative;
        z-index: 1;
    }

    .ww-chat-input-area {
        flex-shrink: 0;
        z-index: 2;
    }

    &__mention-template-editor {
        margin: 16px;
        padding: 16px;
        background: linear-gradient(135deg, #f0f9ff 0%, #e0f2fe 100%);
        border: 2px solid #3b82f6;
        border-radius: 12px;
        box-shadow: 0 4px 6px rgba(59, 130, 246, 0.1);

        &--hidden {
            display: none;
        }
    }

    &__mention-template-header {
        display: flex;
        flex-direction: column;
        gap: 4px;
        margin-bottom: 12px;
        padding-bottom: 12px;
        border-bottom: 1px solid rgba(59, 130, 246, 0.2);
    }

    &__mention-template-title {
        font-size: 14px;
        font-weight: 700;
        color: #1e40af;
        display: flex;
        align-items: center;
        gap: 6px;
    }

    &__mention-template-description {
        font-size: 12px;
        color: #64748b;
        font-weight: 500;
    }

    &__mention-dropzone {
        padding: 16px;
        border: 2px dashed #3b82f6;
        border-radius: 8px;
        background-color: #ffffff;
        min-height: 80px;
        position: relative;
        transition: all 0.2s ease;
        
        &:empty:before {
            content: '👆 Drag WeWeb elements here to design your mention card';
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 13px;
            font-weight: 500;
            color: #94a3b8;
            min-height: 80px;
            text-align: center;
        }
        
        &:hover {
            border-color: #2563eb;
            background-color: rgba(59, 130, 246, 0.02);
        }
    }
}
</style>
