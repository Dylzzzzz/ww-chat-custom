# Implementation Plan

- [x] 1. Add mention system configuration properties to ww-config.js
  - Add mentionEnabled property to enable/disable mention system
  - Add mentionRecordTypes array property for binding record type data
  - Add mapping properties for record type fields (id, label, icon, records)
  - Add mapping properties for record fields (id, label, avatar, metadata)
  - Add mappingMessageMentions property for extracting mentions from incoming messages
  - Group properties in "Mention System" collapsible section in editor
  - _Requirements: 1.1, 4.1, 4.2, 4.3, 4.4, 4.5_

- [x] 2. Add mention button styling properties to ww-config.js
  - Add mentionIcon, mentionIconColor, mentionIconSize properties
  - Add mentionButtonBgColor, mentionButtonHoverBgColor properties
  - Add mentionButtonBorder, mentionButtonBorderRadius, mentionButtonSize properties
  - Add mentionButtonBoxShadow property
  - Group properties in "Mention Button" collapsible section in customStylePropertiesOrder
  - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5_

- [x] 3. Add mention dropdown styling properties to ww-config.js
  - Add mentionDropdownBgColor, mentionDropdownBorder, mentionDropdownBorderRadius properties
  - Add mentionDropdownBoxShadow, mentionDropdownMaxHeight, mentionDropdownMaxWidth properties
  - Add mentionTabTextColor, mentionTabActiveColor, mentionTabHoverColor properties
  - Add mentionRecordBgColor, mentionRecordHoverBgColor, mentionRecordTextColor properties
  - Add mentionRecordFontSize property
  - Group properties in "Mention Dropdown" collapsible section in customStylePropertiesOrder
  - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5_

- [x] 4. Create MentionButton.vue component
  - Create new file src/components/MentionButton.vue
  - Define props for icon, colors, sizes, and styling
  - Implement button template with icon rendering using wwLib.useIcons
  - Add click event emission
  - Add disabled state handling
  - Style button with hover effects and transitions matching attachment button
  - _Requirements: 1.1, 8.1, 8.2, 8.3, 8.4, 8.5_

- [x] 5. Create MentionDropdown.vue component
  - Create new file src/components/MentionDropdown.vue
  - Define props for visibility, recordTypes, activeTab, filterText, highlightedIndex, position
  - Define props for all styling properties
  - Implement tabs row template with click handlers
  - Implement scrollable records list template
  - Add empty state message when no records available
  - Add "No results found" message when filter returns no matches
  - Implement click-outside detection to close dropdown
  - Style dropdown with positioning, shadows, and transitions
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 5.1, 5.2, 5.3, 5.4, 5.5, 10.1, 10.2, 10.3, 10.4, 10.5_

- [x] 6. Add mention state and methods to InputArea.vue
  - Import MentionButton and MentionDropdown components
  - Add reactive state: mentionDropdownVisible, activeMentionTab, mentionFilterText, highlightedRecordIndex, cursorPosition, pendingMentions
  - Implement detectMentionTrigger method to detect @ after space or at start
  - Implement openMentionDropdown method to show dropdown and initialize state
  - Implement closeMentionDropdown method to hide dropdown and reset state
  - Implement insertMention method to replace @ with mention text and store mention data
  - Implement handleMentionKeyboard method for arrow keys, Enter, and Escape
  - Implement handleMentionButtonClick method to open dropdown at cursor position
  - Implement handleTabChange method to switch active tab
  - Implement handleRecordSelect method to insert selected mention
  - Add computed property for filteredRecords based on activeTab and filterText
  - _Requirements: 1.5, 2.1, 2.2, 2.3, 2.4, 2.5, 9.1, 9.2, 9.3, 9.4, 9.5, 12.1, 12.2, 12.3, 12.4, 12.5_

- [x] 7. Update InputArea.vue template to include mention button and dropdown
  - Add MentionButton component before text input (after attachment button if present)
  - Adjust input container flex layout to accommodate mention button
  - Add MentionDropdown component positioned above input area
  - Pass all necessary props to MentionButton (icon, colors, sizes, disabled state)
  - Pass all necessary props to MentionDropdown (visibility, recordTypes, activeTab, etc.)
  - Bind event handlers for mention button click and dropdown events
  - Add @input listener to textarea for detectMentionTrigger
  - Add @keydown listener to textarea for handleMentionKeyboard
  - Update input field width calculation to account for mention button
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 2.4, 10.1, 10.2_

- [x] 8. Update InputArea.vue sendMessage method to include mentions
  - Modify sendMessage to include pendingMentions array in message object
  - Clear pendingMentions array after sending message
  - Ensure mentions array is included in messageSent event payload
  - _Requirements: 7.1, 7.2, 7.3_

- [x] 9. Add mention processing to wwElement.vue
  - Add computed properties for processing mentionRecordTypes with mapping formulas
  - Implement resolveRecordTypeMapping helper function
  - Implement resolveRecordMapping helper function
  - Process incoming messages to extract mentions using mappingMessageMentions
  - Add mentions array to message objects in messages computed property
  - Ensure mentions data is preserved when mapping message fields
  - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 7.4, 7.5_

- [x] 10. Pass mention props from wwElement.vue to InputArea.vue
  - Pass mentionEnabled prop to InputArea
  - Pass processed mentionRecordTypes to InputArea
  - Pass all mention button styling props to InputArea
  - Pass all mention dropdown styling props to InputArea
  - Add conditional rendering of mention button based on mentionEnabled
  - _Requirements: 1.1, 4.1, 4.2_

- [x] 11. Create MentionDropzone.vue component
  - Create new file src/components/MentionDropzone.vue
  - Define props for mention data, recordTypeId, isOwnMessage
  - Implement dropzone using WeWeb dropzone pattern (wwLib.wwElement.useDropzone)
  - Provide mention data and message context to dropzone via local context
  - Implement default fallback rendering when no custom component in dropzone
  - Style dropzone container with appropriate spacing and alignment
  - Add conditional styling based on isOwnMessage prop
  - _Requirements: 6.1, 6.2, 6.3, 6.4, 7.5_

- [x] 12. Update MessageItem.vue to render mention dropzones
  - Import MentionDropzone component
  - Add computed property mentionsByType to group mentions by recordTypeId
  - Add mentions section in template after attachments
  - Render MentionDropzone for each mention in message
  - Pass mention data, recordTypeId, and isOwnMessage to each dropzone
  - Add conditional rendering to only show mentions section when mentions exist
  - Style mentions container with appropriate spacing
  - _Requirements: 6.1, 6.5, 7.5, 11.4_

- [x] 13. Update MessageList.vue to pass mentions to MessageItem
  - Ensure mentions array is passed through to MessageItem components
  - No template changes needed (mentions already part of message object)
  - _Requirements: 7.5_

- [x] 14. Add mention system to chat local context
  - Update chatData computed property in wwElement.vue to include mention configuration
  - Add mentionRecordTypes to utils section of local context
  - Update chatMarkdown documentation to describe mention data structure
  - Ensure mention data is available to custom components in dropzones
  - _Requirements: 7.5_

- [x] 15. Add mention-related trigger events to ww-config.js
  - Add mentionSelected event with recordTypeId, recordId, recordLabel payload
  - Add mentionDropdownOpened event
  - Add mentionDropdownClosed event
  - Document event payloads in triggerEvents section
  - _Requirements: 2.5, 3.5_

- [x] 16. Implement mention filtering logic in MentionDropdown.vue
  - Add computed property filteredRecords that filters based on filterText
  - Implement case-insensitive matching against record label
  - Update highlightedIndex when filtered list changes
  - Reset highlightedIndex to 0 when filter text changes
  - Reset highlightedIndex to 0 when tab changes
  - _Requirements: 12.1, 12.2, 12.3, 12.4, 12.5_

- [x] 17. Implement keyboard navigation in MentionDropdown.vue
  - Add watcher for highlightedIndex to scroll highlighted item into view
  - Implement scrollIntoView for highlighted record element
  - Add keyboard event listeners for arrow keys
  - Ensure highlighted item is always visible in scrollable list
  - _Requirements: 9.1, 9.2, 9.3_

- [x] 18. Add dynamic dropzone registration for mention record types
  - In wwElement.vue, dynamically generate dropzone configurations based on mentionRecordTypes
  - Register dropzones using wwLib.wwElement.useDropzone for each record type
  - Ensure dropzone IDs follow pattern: `mention-${recordTypeId}`
  - Update dropzone labels to use record type labels
  - _Requirements: 6.1, 6.2_

- [x] 19. Style MentionButton to match existing button styles
  - Ensure MentionButton uses same styling patterns as attachment button
  - Add hover and active state transitions
  - Ensure button aligns vertically with other input area buttons
  - Test button appearance with various size and color configurations
  - _Requirements: 1.2, 8.1, 8.2, 8.3, 8.4, 8.5_

- [x] 20. Style MentionDropdown for responsive behavior
  - Ensure dropdown width adapts to content up to maxWidth
  - Implement responsive positioning for small viewports
  - Add smooth transitions for dropdown open/close
  - Ensure dropdown is accessible on mobile devices
  - Test dropdown appearance at various viewport sizes
  - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 10.3, 10.4_

- [x] 21. Add error handling for invalid mention configurations
  - Validate mentionRecordTypes is an array in wwElement.vue
  - Filter out record types without required id and label fields
  - Log warnings in editor mode for invalid configurations
  - Hide mention button when no valid record types exist
  - Provide helpful error messages in console for developers
  - _Requirements: 4.1, 4.2, 4.3_

- [x] 22. Implement mention text extraction for filter
  - In InputArea.vue, extract text after @ symbol for filtering
  - Update mentionFilterText as user types after @
  - Clear filter text when mention is inserted
  - Handle backspace to update filter text
  - Close dropdown if user deletes @ symbol
  - _Requirements: 2.3, 12.1, 12.2, 12.3, 12.4_

- [x] 23. Add support for multiple mentions in single message
  - Ensure pendingMentions array can hold multiple mentions
  - Track each mention insertion separately
  - Preserve all mentions when sending message
  - Render all mentions in message bubble
  - Test with 3+ mentions in single message
  - _Requirements: 11.1, 11.2, 11.3, 11.4, 11.5_

- [x] 24. Update ww-config.js property visibility conditions
  - Hide mention button styling properties when mentionEnabled is false
  - Hide mention dropdown styling properties when mentionEnabled is false
  - Hide mention mapping properties when mentionEnabled is false
  - Show helpful hint text when mentionEnabled is false
  - _Requirements: 4.1_

- [x] 25. Add default mention icon using wwLib.useIcons
  - Implement default at-sign icon SVG in MentionButton.vue
  - Use wwLib.useIcons to load custom icon if specified
  - Fallback to default icon if custom icon fails to load
  - Match icon loading pattern used for send and attachment icons
  - _Requirements: 8.5_

- [x] 26. Implement mention dropdown positioning logic
  - Calculate dropdown position relative to input area
  - Ensure dropdown appears above input area
  - Handle edge cases where dropdown extends beyond viewport
  - Adjust dropdown position if it would be cut off
  - Add positioning tests for various container sizes
  - _Requirements: 10.1, 10.2, 10.3, 10.4, 10.5_

- [x] 27. Add accessibility attributes to mention components
  - Add aria-label to MentionButton
  - Add role="listbox" to MentionDropdown
  - Add aria-expanded to mention button based on dropdown state
  - Add aria-selected to highlighted record in dropdown
  - Add aria-activedescendant to track highlighted item
  - Ensure keyboard navigation is screen reader friendly
  - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5_

- [x] 28. Update component documentation in ww-config.js
  - Add propertyHelp tooltips for all mention properties
  - Add bindingValidation for all mention properties
  - Document mention data structure in property descriptions
  - Add examples for mentionRecordTypes data structure
  - Document dropzone usage for custom mention rendering
  - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5_

- [x] 29. Test mention system with sample data
  - Create sample mentionRecordTypes data with 2-3 record types
  - Test mention button click flow
  - Test @ symbol detection flow
  - Test keyboard navigation through records
  - Test mention insertion and message sending
  - Test message rendering with mentions
  - Test custom component in mention dropzone
  - _Requirements: 1.5, 2.5, 3.5, 9.5, 11.5_

- [x] 30. Add CSS transitions and animations
  - Add fade-in animation for dropdown appearance
  - Add smooth highlight transition for keyboard navigation
  - Add hover effects for tabs and records
  - Add active state styling for selected tab
  - Ensure animations are performant and smooth
  - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5_
