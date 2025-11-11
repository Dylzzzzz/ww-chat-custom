# Requirements Document

## Introduction

This feature enhances the WeWeb chat component to support @mentioning record types with custom interactive message rendering. Users can mention records (e.g., tasks, projects, contacts) directly in chat messages, and these mentions can be rendered as custom interactive components within message bubbles. The system provides a dropdown selector for choosing records to mention and supports dropzones for custom WeWeb component rendering of mentioned records.

## Glossary

- **Chat Component**: The WeWeb custom element that displays messages and handles user input
- **Input Area**: The text input field where users type messages
- **Mention Trigger**: The @ symbol that activates the mention dropdown
- **Mention Dropdown**: A popup interface displaying record types and records available for mentioning
- **Record Type**: A category of mentionable items (e.g., "Tasks", "Projects", "Contacts")
- **Record**: An individual item that can be mentioned (e.g., a specific task or project)
- **Mention Button**: A dedicated button next to the attachment button that opens the mention dropdown
- **Message Bubble**: The visual container displaying a chat message
- **Dropzone**: A WeWeb framework feature allowing custom component placement within a predefined area
- **Mention Data**: The structured data representing a mentioned record within a message
- **Interactive Message**: A message containing mentioned records rendered as custom components

## Requirements

### Requirement 1: Mention Button in Input Area

**User Story:** As a chat user, I want a dedicated @ button next to the attachment button, so that I can easily access the mention functionality without typing.

#### Acceptance Criteria

1. WHEN the Chat Component renders the Input Area, THE Input Area SHALL display a mention button to the left of the text input field
2. WHEN the mention button is present, THE text input field SHALL reduce its width to accommodate the mention button without overlapping
3. WHEN the allowAttachments property is true, THE mention button SHALL be positioned between the attachment button and the text input field
4. WHEN the allowAttachments property is false, THE mention button SHALL be positioned as the leftmost element before the text input field
5. WHEN a user clicks the mention button, THE Chat Component SHALL display the mention dropdown above the Input Area

### Requirement 2: Automatic Mention Detection

**User Story:** As a chat user, I want the mention dropdown to appear automatically when I type @ after a space, so that I can quickly mention records while typing naturally.

#### Acceptance Criteria

1. WHEN a user types the @ character after a space character in the text input, THE Chat Component SHALL display the mention dropdown
2. WHEN a user types the @ character at the beginning of the text input, THE Chat Component SHALL display the mention dropdown
3. WHEN a user types the @ character immediately after another alphanumeric character, THE Chat Component SHALL NOT display the mention dropdown
4. WHEN the mention dropdown is displayed via typing, THE Chat Component SHALL position the dropdown above the Input Area
5. WHEN a user selects a record from the dropdown, THE Chat Component SHALL insert the mention at the cursor position

### Requirement 3: Mention Dropdown Structure

**User Story:** As a chat user, I want to see a dropdown with tabs for different record types and a list of records within each tab, so that I can easily find and select the record I want to mention.

#### Acceptance Criteria

1. WHEN the mention dropdown is displayed, THE mention dropdown SHALL render a row of tabs at the top representing available record types
2. WHEN the mention dropdown is displayed, THE mention dropdown SHALL render a scrollable list of records below the tabs
3. WHEN a user clicks a tab, THE mention dropdown SHALL display the records associated with that record type
4. WHEN no records are available for a record type, THE mention dropdown SHALL display an empty state message
5. WHEN a user clicks a record in the list, THE Chat Component SHALL insert the mention into the message and close the dropdown

### Requirement 4: Bindable Record Type Configuration

**User Story:** As a developer, I want to bind record types and their records to component properties, so that I can dynamically configure what can be mentioned based on my application data.

#### Acceptance Criteria

1. THE Chat Component SHALL provide a bindable property named mentionRecordTypes that accepts an array of record type objects
2. WHEN mentionRecordTypes is bound to an array, THE Chat Component SHALL render tabs in the mention dropdown for each record type
3. THE Chat Component SHALL provide mapping properties for record type fields including id, label, and icon
4. THE Chat Component SHALL provide a bindable property for each record type that accepts an array of record objects
5. THE Chat Component SHALL provide mapping properties for record fields including id, label, avatar, and metadata

### Requirement 5: Mention Dropdown Styling Configuration

**User Story:** As a developer, I want to customize the appearance of the mention dropdown, so that it matches my application's design system.

#### Acceptance Criteria

1. THE Chat Component SHALL provide style properties for mention dropdown background color
2. THE Chat Component SHALL provide style properties for mention dropdown border and border radius
3. THE Chat Component SHALL provide style properties for tab text color, active tab color, and tab hover color
4. THE Chat Component SHALL provide style properties for record item background color and hover color
5. THE Chat Component SHALL provide style properties for mention dropdown maximum height and width

### Requirement 6: Message Dropzones for Custom Rendering

**User Story:** As a developer, I want to add dropzones to message bubbles for each record type, so that I can render custom WeWeb components for mentioned records.

#### Acceptance Criteria

1. WHEN a message contains mention data, THE Message Bubble SHALL provide a dropzone for each configured record type
2. THE Chat Component SHALL pass the mention data to the dropzone as context
3. WHEN a dropzone contains custom components, THE Message Bubble SHALL render those components within the message bubble
4. WHEN no custom components are placed in a dropzone, THE Message Bubble SHALL render a default text representation of the mention
5. THE dropzone SHALL be conditionally visible based on whether the message contains mentions of that record type

### Requirement 7: Mention Data Structure

**User Story:** As a developer, I want mentioned records to be included in message data with a clear structure, so that I can process and render mentions consistently.

#### Acceptance Criteria

1. WHEN a user sends a message with mentions, THE Chat Component SHALL include a mentions array in the message object
2. THE mentions array SHALL contain objects with recordTypeId, recordId, recordLabel, and recordData properties
3. WHEN the messageSent event is triggered, THE event payload SHALL include the complete message object with mentions
4. THE Chat Component SHALL provide mapping properties to extract mentions from incoming message data
5. WHEN rendering messages, THE Chat Component SHALL make mention data available to dropzone components via local context

### Requirement 8: Mention Button Styling Configuration

**User Story:** As a developer, I want to customize the appearance of the mention button, so that it matches the styling of other input buttons.

#### Acceptance Criteria

1. THE Chat Component SHALL provide a style property for mention button background color
2. THE Chat Component SHALL provide a style property for mention button hover background color
3. THE Chat Component SHALL provide a style property for mention button border and border radius
4. THE Chat Component SHALL provide a style property for mention button size
5. THE Chat Component SHALL provide properties for mention icon, mention icon color, and mention icon size

### Requirement 9: Keyboard Navigation in Mention Dropdown

**User Story:** As a chat user, I want to navigate the mention dropdown using keyboard arrows and select records with Enter, so that I can mention records efficiently without using the mouse.

#### Acceptance Criteria

1. WHEN the mention dropdown is open, THE Chat Component SHALL highlight the first record in the list
2. WHEN a user presses the down arrow key, THE Chat Component SHALL move the highlight to the next record
3. WHEN a user presses the up arrow key, THE Chat Component SHALL move the highlight to the previous record
4. WHEN a user presses the Enter key with a record highlighted, THE Chat Component SHALL insert that mention and close the dropdown
5. WHEN a user presses the Escape key, THE Chat Component SHALL close the mention dropdown without inserting a mention

### Requirement 10: Mention Dropdown Positioning

**User Story:** As a chat user, I want the mention dropdown to appear above the input area without covering my typed text, so that I can see both my message and the mention options simultaneously.

#### Acceptance Criteria

1. WHEN the mention dropdown is triggered, THE Chat Component SHALL position the dropdown directly above the Input Area
2. THE mention dropdown SHALL align with the left edge of the Input Area
3. WHEN the mention dropdown height exceeds available space above the Input Area, THE mention dropdown SHALL display a scrollbar
4. THE mention dropdown SHALL have a maximum height property that is bindable
5. WHEN the mention dropdown is open and a user clicks outside the dropdown, THE Chat Component SHALL close the dropdown

### Requirement 11: Multiple Mentions in Single Message

**User Story:** As a chat user, I want to mention multiple records in a single message, so that I can reference multiple items in one conversation.

#### Acceptance Criteria

1. WHEN a user inserts a mention, THE Chat Component SHALL allow the user to continue typing after the mention
2. WHEN a user triggers the mention dropdown again in the same message, THE Chat Component SHALL display the dropdown for a new mention
3. WHEN a message contains multiple mentions, THE mentions array SHALL contain all mentioned records in order
4. WHEN rendering a message with multiple mentions, THE Message Bubble SHALL render all mention dropzones with their respective data
5. THE Chat Component SHALL support an unlimited number of mentions per message

### Requirement 12: Mention Filtering and Search

**User Story:** As a chat user, I want to filter records in the mention dropdown by typing after the @ symbol, so that I can quickly find the record I want to mention.

#### Acceptance Criteria

1. WHEN a user types characters after the @ symbol, THE Chat Component SHALL filter the displayed records based on the typed text
2. THE filtering SHALL match against the record label property
3. WHEN no records match the filter text, THE mention dropdown SHALL display a "No results found" message
4. WHEN a user deletes filter text, THE mention dropdown SHALL update to show matching records
5. WHEN a user switches tabs while filtering, THE filter text SHALL apply to the new tab's records
