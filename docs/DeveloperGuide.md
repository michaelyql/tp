---
  layout: default.md
  title: "Developer Guide"
  pageNav: 3
---

# LegacyLink Developer Guide

<!-- * Table of Contents -->
<page-nav-print />

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

This project is based on the AddressBook-Level3 project created by the [SE-EDU initiative](https://se-education.org).

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

### Architecture

<puml src="diagrams/ArchitectureDiagram.puml" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of classes [`Main`](https://github.com/AY2425S1-CS2103T-T10-4/tp/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/AY2425S1-CS2103T-T10-4/tp/tree/master/src/main/java/seedu/address/MainApp.java)) is in charge of the app launch and shut down.
* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<puml src="diagrams/ArchitectureSequenceDiagram.puml" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<puml src="diagrams/ComponentManagers.puml" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/AY2425S1-CS2103T-T10-4/tp/tree/master/src/main/java/seedu/address/ui/Ui.java)

<puml src="diagrams/UiClassDiagram.puml" alt="Structure of the UI Component"/>

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/AY2425S1-CS2103T-T10-4/tp/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/AY2425S1-CS2103T-T10-4/tp/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/AY2425S1-CS2103T-T10-4/tp/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<puml src="diagrams/LogicClassDiagram.puml" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API call as an example.

<puml src="diagrams/DeleteSequenceDiagram.puml" alt="Interactions Inside the Logic Component for the `delete 1` Command" />

<box type="info" seamless>

**Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</box>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a person).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take several interactions (between the command object and the `Model`) to achieve.
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<puml src="diagrams/ParserClasses.puml" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/AY2425S1-CS2103T-T10-4/tp/tree/master/src/main/java/seedu/address/model/Model.java)

<puml src="diagrams/ModelClassDiagram.puml" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)

<box type="info" seamless>

**Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Person` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Person` needing their own `Tag` objects.<br>

<puml src="diagrams/BetterModelClassDiagram.puml" width="450" />

</box>


### Storage component

**API** : [`Storage.java`](https://github.com/AY2425S1-CS2103T-T10-4/tp/tree/master/src/main/java/seedu/address/storage/Storage.java)

<puml src="diagrams/StorageClassDiagram.puml" width="550" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.address.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### \[Proposed\] Undo/redo feature

#### Proposed Implementation

The proposed undo/redo mechanism is facilitated by `VersionedAddressBook`. It extends `AddressBook` with an undo/redo history, stored internally as an `addressBookStateList` and `currentStatePointer`. Additionally, it implements the following operations:

* `VersionedAddressBook#commit()` — Saves the current address book state in its history.
* `VersionedAddressBook#undo()` — Restores the previous address book state from its history.
* `VersionedAddressBook#redo()` — Restores a previously undone address book state from its history.

These operations are exposed in the `Model` interface as `Model#commitAddressBook()`, `Model#undoAddressBook()` and `Model#redoAddressBook()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `VersionedAddressBook` will be initialized with the initial address book state, and the `currentStatePointer` pointing to that single address book state.

<puml src="diagrams/UndoRedoState0.puml" alt="UndoRedoState0" />

Step 2. The user executes `delete 5` command to delete the 5th person in the address book. The `delete` command calls `Model#commitAddressBook()`, causing the modified state of the address book after the `delete 5` command executes to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

<puml src="diagrams/UndoRedoState1.puml" alt="UndoRedoState1" />

Step 3. The user executes `add n/David …​` to add a new person. The `add` command also calls `Model#commitAddressBook()`, causing another modified address book state to be saved into the `addressBookStateList`.

<puml src="diagrams/UndoRedoState2.puml" alt="UndoRedoState2" />

<box type="info" seamless>

**Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.

</box>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

<puml src="diagrams/UndoRedoState3.puml" alt="UndoRedoState3" />


<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</box>

The following sequence diagram shows how an undo operation goes through the `Logic` component:

<puml src="diagrams/UndoSequenceDiagram-Logic.puml" alt="UndoSequenceDiagram-Logic" />

<box type="info" seamless>

**Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</box>

Similarly, how an undo operation goes through the `Model` component is shown below:

<puml src="diagrams/UndoSequenceDiagram-Model.puml" alt="UndoSequenceDiagram-Model" />

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</box>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`. Thus, the `addressBookStateList` remains unchanged.

<puml src="diagrams/UndoRedoState4.puml" alt="UndoRedoState4" />

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

<puml src="diagrams/UndoRedoState5.puml" alt="UndoRedoState5" />

The following activity diagram summarizes what happens when a user executes a new command:

<puml src="diagrams/CommitActivityDiagram.puml" width="250" />

#### Design considerations:

**Aspect: How undo & redo executes:**

* **Alternative 1 (current choice):** Saves the entire address book.
  * Pros: Easy to implement.
  * Cons: May have performance issues in terms of memory usage.

* **Alternative 2:** Individual command knows how to undo/redo by
  itself.
  * Pros: Will use less memory (e.g. for `delete`, just save the person being deleted).
  * Cons: We must ensure that the implementation of each individual command are correct.

_{more aspects and alternatives to be added}_

### \[Proposed\] Data archiving

_{Explain here how the data archiving feature will be implemented}_


--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

**Target user profile**:

* Has a large extended family with numerous contacts to manage
* Values maintaining connections with various family branches (direct family, paternal side, maternal side, in-laws)
* Organizes or participates in frequent family gatherings and events
* Desires an efficient way to keep family contact information up-to-date
* Appreciates tools that help navigate complex family dynamics
* Is comfortable using digital tools for personal organization
* Ranges from young adults to seniors who want to stay connected with their extended family

**Value proposition**: LegacyLink offers a comprehensive family contact management system that simplifies the organization of large family trees, streamlines event planning, and helps maintain family connections more effectively than traditional contact management methods.
It revolutionizes the "family experience"!
### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​         | I want to …​                                                 | So that I can…​                                                       |
|----------|-----------------|--------------------------------------------------------------|-----------------------------------------------------------------------|
| `* * *`  | As a user       | add contact information of the family member                 | I can retrieve their contact information                              |
| `* * *`  | As a user       | add the relationship of the family members                   | I can know the relationship between people                            |
| `* * *`  | As a user       | update the information of family members in the contact list | I can keep the latest information of my family                        |
| `* * *`  | As a user       | add events tied to family members (e.g birthdays)            | I can set reminders on that date so that I don't ever forget about it |
| `* * *`  | As an organizer | track RSVPs and attendance for each event                    | I know who is attending the event and can plan accordingly            |
| `* * *`  | As an organizer | schedule family events                                       | I can plan and coordinate events                                      |
| `* * *`  | As an organizer | see the contact list of family members                       | I know whose contacts that I have not added yet and add them          |
| `* * *`  | As an organizer | update the event's information after creating it             | attendees can see the updated event details                           |
| `* * *`  | As an organizer | delete an event                                              | I can cancel an event                                                 |



*{More to be added}*

### Use cases

(For all use cases below, the **System** is the `LegacyLink` and the **Actor** is the `user`, unless specified otherwise)

**Use case 1: Add contact**

**MSS**
1. User enters name, phone number, email and relationship of the contact.
2. User confirms details of the contact.
3. System adds the contact.

    Use case ends.

**Extensions**

* 1a. User enters an invalid name, phone number, email, and/or relationship.

    Use case resumes at step 1.


**Use case 2: Delete contact**

**MSS**

1.  User views all contacts (UC-3).
2.  System shows a list of persons.
3.  User requests to delete a specific contact in the list.
4.  System deletes the person.

    Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.


* 3a. The given index is invalid.

    * 3a1. System shows an error message.

      Use case resumes at step 2.

**Use case 3: View all contacts**

**MSS**

1. User requests to view the list of contacts.
2. The system displays a list of all contacts.
3. User can scroll through the list to see all the contact listed.
4. User can click on a contact to view more details.

    Use case ends.

**Use case 4: Update information of contact**

**MSS**

1. User lists all contacts (UC-3).
2. User selects contact to update.
3. User can edit name / phone number / email / relationship of contact.
4. System registers the changes.

    Use case ends.

**Extensions**

* 2a. The given index is invalid.

    * 1a1. System shows an error message.

      Use case resumes at step 2.


* 3a. The changed details are invalid.

  * 3a1. System shows an error message.

    Use case resumes at step 3.

*a. At anytime, User can choose to cancel updating the contact.

    *a1. System does not update any contact details.
    Use case ends.


**Use case 5: Add event**

**MSS**

1. User enters name, start date, end date, location of the event.
2. User confirms the details of the event.
3. System adds the event.
4. User is given feedback that the event is added successfully.

    Use case ends.

**Extensions**

* 1a. User enters index of persons attending.

    Use case resumes at step 2.


* 1b. The inputs are invalid

    * 1b1. System shows an error message

        Use case resumes at step 1.


*a At any time, Users chooses to cancel the adding.


**Use case 6: Delete an event**

**Preconditions:**

1. Event list must have at least one event.


**MSS**

1. User lists all events(UC-7).
2. System shows a list of events.
3. User selects an event to delete.
4. User confirms their intention and the event is deleted.
5. User is given feedback that the event is deleted successfully.
6. User no longer sees the event in the event list.

    Use case ends.

**Extensions**

* 3a. The given index is invalid.

    * 3a1. System shows an error message.

      Use case resumes at step 3.


* 4a. If the User cancels deleting the event, the event is kept and the use case ends.

*a. If the user exits the application without confirming, the event is kept and the use case ends.

**Use case 7: View all events**

**MSS**

1. User lists all events.
2. The system displays a list of all events.
3. User can scroll through the list to see all the events listed.
4. User can click on an event to view more details.

    Use case ends

**Use case 8: Update event information**

**MSS**

1. User views all events(UC-7).
2. User selects event to edit.
3. User changes the relevant event details.
4. User saves the information.

    Use case ends.

**Extensions**

* 2a. The given index is invalid.

    * 2a1. System shows an error message.

      Use case resumes at step 2.


* 3a. User does not change event details.

    Use case resumes at step 2.


* 3b. The changed details are invalid.

    * 3b1. System shows an error message.

        Use case resumes at step 3.


*a At any time, User chooses to cancel the edit.

*{More to be added}*

### Non-Functional Requirements

1.  Should work on any _mainstream OS_ as long as it has Java `17` or above installed.
2.  Should be able to hold up to 1000 persons without a noticeable sluggishness in performance for typical usage.
3.  A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.

*{More to be added}*

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, MacOS
* **Private contact detail**: A contact detail that is not meant to be shared with others

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<box type="info" seamless>

**Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</box>

### Launch and shutdown

1. Initial launch

   1. Download the jar file and copy into an empty folder

   1. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

1. Saving window preferences

   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

1. _{ more test cases …​ }_

### Listing all persons

1. List all persons

    1. Test case: `list -p` (must be an exact match)
        Expected: `Listed all person` is shown in the status message. Tabs switched to Contacts.

### Adding a person

1. Adding a person

    1. Prerequisites: List all persons using the `list -p` command. Multiple persons in the list.

    1. Test case: `add -n John Doe -p 98765432 -e johnd@example.com -rs Brother`<br>
       Expected: Person with name `John Doe` with phone number `98765432`, email `johnd@example.com` and relationship of `Brother` is added. `New person added: John Doe; Phone: 98765432; Email: johnd@example.com; Relationship: Brother` is shown in the status message.

    1. Test case: `add`<br>
       Expected: No person is added to the contact. Error details shown in the status message. Status bar remains the same.

    1. Test case: `add 0` <br>
       Expected: No person is added to the contact. Error details shown in the status message. Status bar remains the same.

   1. Test case (Invalid name): `add -n John_Doe -p 98765432 -e johnd@example.com -rs Brother` <br>
      Expected: No person is added to the contact. Error details shown in the status message. Status bar remains the same.

   1. Test case (Invalid phone number): `add -n John Doe -p 1 -e johnd@example.com -rs Brother` <br>
      Expected: No person is added to the contact. Error details shown in the status message. Status bar remains the same.

   1. Test case (Invalid email): `add -n John Doe -p 98765432 -e johndexample -rs Brother` <br>
      Expected: No person is added to the contact. Error details shown in the status message. Status bar remains the same.

   1. Test case (Invalid relationship): `add -n John Doe -p 98765432 -e johnd@example.com -rs Brother333` <br>
      Expected: No person is added to the contact. Error details shown in the status message. Status bar remains the same.

   1. Test case (Multiple values for fields): `add -n John Doe -p 98765432 -e johnd@example.com -rs Brother -rs Brother` <br>
      Expected: No person is added to the contact. Error details shown in the status message. Status bar remains the same.

   1. Test case (Missing fields): `add -n John Doe -e johnd@example.com -rs Brother` <br>
      Expected: No person is added to the contact. Error details shown in the status message. Status bar remains the same.

2. Adding a duplicate person

    1. Prerequisites: List all persons using the `list -p` command. Person with name `John Doe` with phone number `98765432`, email `johnd@example.com` and relationship of `Brother` already exists.

    1. Test case: `add -n John Doe -p 98765432 -e johnd@example.com -rs Brother`
        Expected: `This person already exists in the address book` is displayed.

### Deleting a person

1. Deleting a person while all persons are being shown

   1. Prerequisites: List all persons using the `list -p` command. Multiple persons in the list.

   1. Test case: `delete 1`<br>
      Expected: First person is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

   1. Test case: `delete 0`<br>
      Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

### Editing a person

1. Editing a person, where result does not cause duplicate persons.

    1. Prerequisites: List all persons using the `list -p` command. Multiple persons in the list.

    1. Test case: `edit 1 -n Bernice Tan`<br>
       Expected: The name of the first person is edited to `Bernice Tan`. Details of the added person shown in the status message.

   1. Test case: `edit 1 -p 123123123`<br>
      Expected: The phone number of the first person is edited to `123123123`. Details of the added person shown in the status message.

   1. Test case: `edit 1 -e johnd@example.com`<br>
      Expected: The email of the first person is edited to `johnd@example.com`. Details of the added person shown in the status message.

   1. Test case: `edit 1 -rs Sister`<br>
      Expected: The name of the first person is edited to `Sister`. Details of the added person shown in the status message. Timestamp in the status bar is updated.

   1. Other correct edit commands to try (more than 1 field edited): `edit 1 -p 12345678 -rs Sister` <br>
       Expected: The phone number and relationship of the first person is edited to `12345678` and `Sister`. Details of the added person shown in the status message.

   1. Test case (No fields provided): `edit 1`<br>
      Expected: No person is edited in the contact. Error details shown in the status message. Status bar remains the same.

   1. Test case (Multiple values for fields): `edit 1 -n John -n Doe`<br>
      Expected: No person is edited in the contact. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect add commands to try: `edit`, `edit x` (where x is larger than the list size), <br>
      Expected: Expected: No person is edited in the contact. Error details shown in the status message. Status bar remains the same.

2. Editing a person, where result causes duplicate persons.

   1. Prerequisites: `add -n Johnny Doe -p 98765432 -e johnd@example.com -rs Brother` followed by `add -n John Doe -p 98765432 -e johnd@example.com -rs Brother`

    1. Test case: `edit 1 -n John Doe`<br>
       Expected: `This person already exists in the address book.` is shown in the status message.

    1. Other test cases to try: editing 1 or more fields to cause a person to have the same `name`, `phone number`, `email` and `relationship` as another person in the list.
       Expected: `This person already exists in the address book.` is shown in the status message.

### Listing all events

1. Listing all events

   1. Test case: `list -e` (must be an exact match)
        Expected: `Listed all events` is shown in the status message. Tabs switched to Events.

### Adding an event

1. Adding an event

    1. Prerequisites: List all events using the `list -e` command. Multiple events in the list. If adding attendees, attendees must exist in the address book.

    1. Test case (No attendees): `event -n Study -sd 2025-01-01 -ed 2025-01-01 -l School`<br>
       Expected: Event with name `Study` with start date `Jan 01 2025`, end date `Jan 01 2025`, location `School` is added. `New event added: Study; Date: 2025-01-01 - 2025-01-01; Location: School; No Attendees.` is shown in the status message.

    1. Test case: `event`<br>
       Expected: No event is added to the contact. Error details shown in the status message. Status bar remains the same.

    1. Test case: `event -n abc` <br>
       Expected: No event is added to the contact. Error details shown in the status message. Status bar remains the same.

    1. Test case (Invalid dates): `event -n Study -sd 2030-01-01 -ed 2025-01-01 -l School` <br>
       Expected: No event is added to the contact. Error details shown in the status message. Status bar remains the same.

    1. Test case (Invalid attendees): `event -n Study -sd 2025-01-01 -ed 2025-01-01 -l School -a -10` <br>
       Expected: No event is added to the contact. Error details shown in the status message. Status bar remains the same.
   
    1. Test case (Multiple values for fields): `event -n Study -n Event -sd 2025-01-01 -ed 2025-01-01 -l School` <br>
       Expected: No event is added to the contact. Error details shown in the status message. Status bar remains the same.

    1. Test case (Missing fields): `event -n Study -sd 2025-01-01 -l School` <br>
       Expected: No event is added to the contact. Error details shown in the status message. Status bar remains the same.

2. Adding a duplicate event

    1. Prerequisites: List all events using the `list -e` command. Event with name `Study` with start date `Jan 01 2025`, end date `Jan 01 2025`, location `School` already exists.

    1. Test case: `event -n Study -sd 2025-01-01 -ed 2025-01-01 -l School`
       Expected: `This event already exists in the event book` is displayed.

### Saving data

1. Dealing with missing/corrupted data files

   1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_

1. _{ more test cases …​ }_

## **Appendix: Effort**

## **Appendix: Planned Enhancements**

Team size: 5

