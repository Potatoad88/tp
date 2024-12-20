---
  layout: default.md
  title: "Developer Guide"
  pageNav: 3
---

# NUStates Developer Guide

<!-- * Table of Contents -->
<page-nav-print />

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

[//]: # (_{ list here sources of all reused/adapted ideas, code, documentation, and third-party libraries -- include links to the original source as well }_)
This project is based on the AddressBook-Level3 project created by the [SE-EDU initiative](https://se-education.org).

Material Community Icons by [MaterialCommunityIcons](https://materialdesignicons.com/).

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

**`Main`** (consisting of classes [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java)) is in charge of the app launch and shut down.
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

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

<puml src="diagrams/UiClassDiagram.puml" alt="Structure of the UI Component"/>

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

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
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<puml src="diagrams/ModelClassDiagram.puml" width="800" />


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

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

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

**Target user profile**: Real Estate Agents

* has a need to manage a significant number of contacts
* prefer desktop apps over other types
* can type fast
* prefers typing to mouse interactions
* is reasonably comfortable using CLI apps

**Value proposition**: organize and categorize client and seller contacts, schedule appointments, contact clients about new property listings that match their client's preferences, and be notified to contact clients on key dates or at regular intervals.


### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​                                                                  | I want to …​                                                       | So that I can…​                                                                                    |
|----------|--------------------------------------------------------------------------|--------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| `* * *`  | Busy Real Estate Agent                                                   | search for contacts fast                                           | save time when trying to contact a person                                                          |
| `* * *`  | Real Estate Agent                                                        | be able to add contacts                                            | contact and find details about a person                                                            |
| `* * *`  | Real Estate Agent                                                        | delete contacts of a client                                        | remove irrelevant clients from my list                                                             |
| `* * *`  | Real Estate Agent                                                        | be able to keep track of all the properties listed                 | help my clients quickly find the best-suited property for them                                     |
| `* * *`  | Real Estate Agent                                                        | search for properties under me                                     | identify the specific property I want to find                                                      |
| `* * *`  | Real Estate Agent                                                        | view all my clients                                                | browse and find clients                                                                            |
| `* *`    | Real Estate Agent                                                        | know my total list of clients who are actively searching for homes | have a full view of all of my clients                                                              |
| `* *`    | Real Estate Agent                                                        | view full details of a particular client                           | view all saved details of the client in one place                                                  |
| `* *`    | Real Estate Agent                                                        | easily update contact details                                      | keep my address book up to date and accurate                                                       |
| `* *`    | Real Estate Agent                                                        | filter contacts based on names                                     | find contacts I am familiar with via name                                                          |
| `* *`    | Real Estate Agent                                                        | store multiple contacts for each client                            | contact them via their most comfortable means                                                      |
| `*`      | Forgetful Real Estate Agent                                              | keep track of the client birthdays                                 | send them well wishes to maintain relations                                                        |
| `*`      | Forgetful Real Estate Agent                                              | remember what my clients preferences are                           | contact them when a new property matching their criteria is available                              |
| `*`      | Busy Real Estate Agent                                                   | be able to mass delete clients                                     | avoid having to do it individually                                                                 |
| `*`      | Forgetful Real Estate Agent who wish to deal with more important clients | categorize different clients/contacts under different priority     | remember which contacts are more important                                                         |
| `*`      | Real Estate Agent                                                        | filter clients based on some specified criteria                    | find clients relevant to what I am looking for                                                     |
| `*`      | Busy Real Estate Agent                                                   | be able to have multiple accounts/phone numbers                    | separate contacts from my work life and personal life                                              |
| `*`      | Systematic Real Estate Agent                                             | be able to tag my contacts under different labels                  | have quick search for people by searching for the tag                                              |
| `*`      | Busy Real Estate Agent                                                   | be able to edit client details and contacts                        | save time when changing the client details without having to delete and recreate them              |
| `*`      | Real Estate Agent                                                        | pin certain clients at the top of my list                          | quickly access those contacts                                                                      |
| `*`      | Real Estate Agent                                                        | be able to undo a command                                          | recover accidentally executed commands                                                             |
| `*`      | Real Estate Agent                                                        | receive validation feedback when entering client details           | avoid entering invalid or incorrect details                                                        |
| `*`      | Real Estate Agent                                                        | be able detect duplicate client contacts                           | avoid having multiple entries for the same client                                                  |
| `*`      | Real Estate Agent                                                        | hide/archive client details                                        | hide completed transactions and past client details, keeping my contact list updated and organized |
| `*`      | Busy Real Estate Agent                                                   | be able to see a list of my most recently added client contacts    | quickly find and access them                                                                       |
| `*`      | Real Estate Agent                                                        | sort the client list                                               | organize the client list based on my preference                                                    |
| `*`      | Real Estate Agent                                                        | view the history of interactions with each contact                 | remember past conversations and follow-up                                                          |
| `*`      | Real Estate Agent                                                        | filter contacts based on their preferred location or address       | find contacts which are in a specific area or region quickly                                       |
| `*`      | Strategic Real Estate Agent                                              | add custom notes to each contact profile                           | record specific details or preferences that they have so that I can personalize my service         |
| `*`      | Real Estate Agent                                                        | set priority levels for different contacts                         | focus more attention on the more urgent/important clients                                          |
| `*`      | Real Estate Agent                                                        | categorize contacts by their preferred language                    | communicate with the client with their preferred language of choice                                |
| `*`      | Real Estate Agent                                                        | tag contacts based on the type of property they are interested in  | quickly match properties to the right clients                                                      |
| `*`      | Real Estate Agent                                                        | segment my contact list by geographic region                       | target my outreach based on specific property locations or markets                                 |

### Use cases

(For all use cases below, the **System** is the `NUStates application` and the **Actor** is the `user`, unless specified otherwise)


**Use case: UC01 - Add a contact to address book**

**MSS**

1.  User requests to add a new contact by providing the required details.
2.  NUStates validates the provided details.
3.  NUStates adds the new contact to the list.
4.  NUStates shows a success message with the contact details.

    Use case ends.

**Extensions**

* 2a. The provided details are invalid.
    * 2a1. NUStates shows an error message indicating the invalid fields.
    * 2a2. User corrects the invalid details.
    * 2a3. NUStates re-validates the details.

      Use case resumes at step 3.

* 2b. The contact already exists in the NUStates.
    * 2b1. NUStates shows an error message indicating the duplicate contact.

      Use case ends.

* 2c. Required fields are missing.
    * 2c1. NUStates shows an error message indicating the missing fields.
    * 2c2. User provides the missing details.
    * 2c3. NUStates re-validates the details.

      Use case resumes at step 3.

The following activity diagram summarizes the steps involved in adding a contact:

<puml src="diagrams/AddCommandActivityDiagram.puml" alt="AddCommandActivityDiagram" />

**Use case: UC02 - Delete a contact**

**MSS**

1.  User requests to delete a contact by providing the index.
2.  NUStates validates the provided index.
3.  NUStates deletes the contact from the list.
4.  NUStates shows a success message with the deleted contact details.

    Use case ends.

**Extensions**

* 2a. The provided index is missing or not a positive integer.
    * 2a1. NUStates shows an error message indicating the invalid index.

      Use case ends.

* 2b. The provided index is more than the number of contacts in the list.
    * 2b1. NUStates shows an error message indicating the invalid index.

      Use case ends.

**Use case: UC03 - List all contacts**

**MSS**

1.  User requests to list all contacts.
2.  NUStates lists all contacts.
3.  NUStates shows a success message indicating all contacts are listed.

    Use case ends.

**Use case: UC04 - Search for a contact using name**

**MSS**

1.  User requests to search for a contact by providing a keyword related to contact's name.
2.  NUStates validates the provided keyword.
3.  NUStates performs a case-insensitive search for contacts where the name begins with the keyword.
4.  NUStates shows a list of matching contacts with their details.

    Use case ends.

**Extensions**

* 2a. The provided keyword is missing or invalid.
    * 2a1. NUStates shows an error message indicating the invalid keyword.

      Use case ends.

* 3a. No contacts have names that match the provided keyword.
    * 3a1. NUStates shows a message indicating no contacts were found.

      Use case ends.
      
**Use case: UC05 - Search for a contact using phone number**

**MSS**

1.  User requests to search for a contact by providing a keyword related to contact's phone number.
2.  NUStates validates the provided keyword.
3.  NUStates performs the search for contacts where the phone number begins with the keyword.
4.  NUStates shows a list of matching contacts with their details.

    Use case ends.

**Extensions**

* 2a. The provided keyword is missing or invalid.
    * 2a1. NUStates shows an error message indicating the invalid keyword.

      Use case ends.

* 3a. No contacts have phone number that match the provided keyword.
    * 3a1. NUStates shows a message indicating no contacts were found.

      Use case ends.

**Use case: UC06 - Search for a contact using tag assigned to it**

**MSS**

1.  User requests to search for a contact by providing a keyword related to contact's tag.
2.  NUStates validates the provided keyword.
3.  NUStates performs a case-insensitive search for contacts where the tag begins with the keyword.
4.  NUStates shows a list of matching contacts with their details.

    Use case ends.

**Extensions**

* 2a. The provided keyword is missing or invalid.
    * 2a1. NUStates shows an error message indicating the invalid keyword.

      Use case ends.

* 3a. No contacts have tags that match the provided keyword.
    * 3a1. NUStates shows a message indicating no contacts were found.

      Use case ends.

**Use case: UC07 - Search for a property using a keyword**

**MSS**

1.  User requests to search for a property by providing a keyword.
2.  NUStates validates the provided keyword.
3.  NUStates performs a case-insensitive search for properties.
4.  NUStates shows a list of matching contacts with their details.

    Use case ends.

**Extensions**

* 2a. The provided keyword is missing or invalid.
    * 2a1. NUStates shows an error message indicating the invalid keyword.

      Use case ends.

* 3a. No properties match the provided keyword.
    * 3a1. NUStates shows a message indicating no contacts were found.

      Use case ends.

**Use case: UC08 - Pin contact**

**MSS**

1.  User requests to pin a specific contact by providing the index number.
2.  NUStates validates the provided index number.
3.  NUStates pins the contact at the provided index, moving it to the top of the list view.
4.  NUStates shows a confirmation message that the item has been successfully pinned.

    Use case ends.

**Extensions**

* 2a. The provided index number is missing or invalid.
    * 2a1. NUStates shows an error message indicating the invalid index number.

* 3a. The item is already pinned.
    * 3a1. NUStates shows a message indicating that the item is already pinned.

      Use case ends.

**Use case: UC09 - Unpin contact**

**MSS**

1.  User requests to unpin a specific contact by providing the index number
2.  NUStates validates the provided index number.
3.  NUStates unpins the contact at the provided index, moving it back to the bottom of the pinned list.
4.  NUStates shows a confirmation message that the item has been successfully unpinned.

    Use case ends.

**Extensions**

* 2a. The provided index number is missing or invalid.
    * 2a1. NUStates shows an error message indicating the invalid index number.

* 3a. The item is already unpinned.
    * 3a1. NUStates shows a message indicating that the item is already unpinned.

      Use case ends.

**Use case: UC10 - Edit a contact**

**MSS**

1.  User requests to edit an existing contact by providing the required details.
2.  NUStates validates the provided details.
3.  NUStates updates the existing contact in the list with the new details.
4.  NUStates shows a success message with the contact details.

    Use case ends.

**Extensions**

* 2a. The provided details are invalid.
    * 2a1. NUStates shows an error message indicating the invalid fields.
    * 2a2. User corrects the invalid details.
    * 2a3. NUStates re-validates the details.

      Use case resumes at step 3.

**Use case: UC11 - Add a Property-To-Sell**

**MSS**

1.  User requests to add a new `Property-To-Sell` to the specified existing contact by providing the required details.
2.  NUStates validates the provided details.
3.  NUStates adds the new `Property-To-Sell` to the specified existing contact in the list.
4.  NUStates shows a success message with the contact details.

    Use case ends.

**Extensions**

* 2a. The provided details are invalid.
    * 2a1. NUStates shows an error message indicating the invalid fields.
    * 2a2. User corrects the invalid details.
    * 2a3. NUStates re-validates the details.

      Use case resumes at step 3.

* 2b. The existing contact in the NUStates already has the same `Property-To-Sell` in the list.
    * 2b1. NUStates shows an error message indicating the duplicate `Property-To-Sell`.
    * 2b2. User corrects the invalid details.
    * 2b3. NUStates re-validates the details.

      Use case resumes at step 3.

* 2c. Required fields are missing.
    * 2c1. NUStates shows an error message indicating the missing fields.
    * 2c2. User provides the missing details.
    * 2c3. NUStates re-validates the details.

      Use case resumes at step 3.

* 2d. Someone in the NUStates is already wanting to sell the `Property-To-Sell` that the user specified.
    * 2d1. NUStates shows an error message indicating the existing `Property-To-Sell`
    * 2d2. User corrects the invalid details.
    * 2d3. NUStates re-validates the details.

      Use case resumes at step 3.

**Use case: UC12 - Add a Property-To-Buy**

**MSS**

1.  User requests to add a new `Property-To-Buy` to the specified existing contact by providing the required details.
2.  NUStates validates the provided details.
3.  NUStates adds the new `Property-To-Buy` to the specified existing contact in the list.
4.  NUStates shows a success message with the contact details.

    Use case ends.

**Extensions**

* 2a. The provided details are invalid.
    * 2a1. NUStates shows an error message indicating the invalid fields.
    * 2a2. User corrects the invalid details.
    * 2a3. NUStates re-validates the details.

      Use case resumes at step 3.

* 2b. The existing contact in the NUStates already has the same `Property-To-Buy` in the list.
    * 2b1. NUStates shows an error message indicating the duplicate `Property-To-Buy`.
    * 2b2. User corrects the invalid details.
    * 2b3. NUStates re-validates the details.

      Use case resumes at step 3.

* 2c. Required fields are missing.
    * 2c1. NUStates shows an error message indicating the missing fields.
    * 2c2. User provides the missing details.
    * 2c3. NUStates re-validates the details.

      Use case resumes at step 3.

**Use case: UC13 - Delete a Property-To-Sell**

**MSS**

1.  User requests to delete a new `Property-To-Sell` from the specified existing contact by providing the required details.
2.  NUStates validates the provided details.
3.  NUStates deletes the new `Property-To-Sell` from the specified existing contact in the list.
4.  NUStates shows a success message with the contact details.

    Use case ends.

**Extensions**

* 2a. The provided details are invalid.
    * 2a1. NUStates shows an error message indicating the invalid fields.
    * 2a2. User corrects the invalid details.
    * 2a3. NUStates re-validates the details.

      Use case resumes at step 3.

* 2b. The existing contact in the NUStates does not have the specified `Property-To-Sell` in the list.
    * 2b1. NUStates shows an error message indicating the missing `Property-To-Sell`.
    * 2b2. User corrects the invalid details.
    * 2b3. NUStates re-validates the details.

      Use case resumes at step 3.

* 2c. Required fields are missing.
    * 2c1. NUStates shows an error message indicating the missing fields.
    * 2c2. User provides the missing details.
    * 2c3. NUStates re-validates the details.

      Use case resumes at step 3.

**Use case: UC14 - Delete a Property-To-Buy**

**MSS**

1.  User requests to sort contact list by a specific field.
2.  NUStates validates the provided details.
3.  NUStates deletes the new `Property-To-Buy` from the specified existing contact in the list.
4.  NUStates shows a success message with the contact details.

    Use case ends.

**Extensions**

* 2a. The provided details are invalid.
    * 2a1. NUStates shows an error message indicating the invalid fields.
    * 2a2. User corrects the invalid details.
    * 2a3. NUStates re-validates the details.

      Use case resumes at step 3.

* 2b. The existing contact in the NUStates does not have the specified `Property-To-Buy` in the list.
    * 2b1. NUStates shows an error message indicating the missing `Property-To-Buy`.
    * 2b2. User corrects the invalid details.
    * 2b3. NUStates re-validates the details.

      Use case resumes at step 3.

* 2c. Required fields are missing.
    * 2c1. NUStates shows an error message indicating the missing fields.
    * 2c2. User provides the missing details.
    * 2c3. NUStates re-validates the details.

      Use case resumes at step 3.


**Use case: UC15 - Sort contact list**

**MSS**

1.  User requests to sort contact list by a specific field and order.
2. NUStates validates the provided details.
3. NUStates sorts the contact list based on the specified field and order.
4. NUStates shows a success message with the sorted contact list.

    Use case ends.

**Extensions**

* 2a. The provided details are invalid.
    * 2a1. NUStates shows an error message indicating the invalid fields.
    * 2a2. User corrects the invalid details.
    * 2a3. NUStates re-validates the details.

      Use case resumes at step 3.

* 2b. Required fields are missing.
    * 2b1. NUStates shows an error message indicating the missing fields.
    * 2b2. User provides the missing details.
    * 2b3. NUStates re-validates the details.

      Use case resumes at step 3.

**Use case: UC16 - Sort property lists in a specific contact**

**MSS**

1.  User requests to sort property lists in a specific contact by a specific field and order.
2. NUStates validates the provided details.
3. NUStates sorts the property lists for that contact based on the specified field and order.
4. NUStates shows a success message with the sorted property lists.

   Use case ends.

**Extensions**

* 2a. The provided details are invalid.
    * 2a1. NUStates shows an error message indicating the invalid fields.
    * 2a2. User corrects the invalid details.
    * 2a3. NUStates re-validates the details.

      Use case resumes at step 3.

* 2b. Required fields are missing.
    * 2b1. NUStates shows an error message indicating the missing fields.
    * 2b2. User provides the missing details.
    * 2b3. NUStates re-validates the details.

      Use case resumes at step 3.

**Use case: UC17 - View statistics**

**MSS**

1.  User requests to view statistics of the contacts.
2.  NUStates shows the statistics of the contacts.

    Use case ends.

### Non-Functional Requirements

1.  Should work on any mainstream OS as long as it has Java `17` or above installed.
2.  Should be able to hold up to 1000 persons without a noticeable sluggishness in performance for typical usage.
3.  A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.
4.  Client searches should provide results even with partial or incomplete input and return results instantly (within 1 second).
5.  The system should be available 99.9% of the time to ensure real-estate agents can access it whenever they need to.
6.  The system should gracefully handle failures, ensuring no data loss during system crashes or unexpected outages.
7.  All code should be well-documented to ensure future developers can easily understand and modify the application.

*{More to be added}*

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, macOS
* **Person**: An entity representing a client who can buy properties, sell properties or do both. It contains personal details such as name, phone number, email and address. Contact can also include preferences like seller or buyer ranges and associated properties.
* **Property**: A real estate asset associated to a contact. Each property has details such as name, address, price, number of rooms and optional tags(property type/condition/ownership type, etc.)
* **Index**: A numerical identifier used to refer to a specific contact or property in the system. These are used for operations like deletion, updating, or viewing details.
* **Tag**: A label used to categorize contacts or properties. Examples can include "Investor", "Luxury", "Renter", etc. They are used to provide additional context and for easy filtering and searching.
* **Selling Price**: The price in which the seller is willing to sell the property.
* **Buying Price**: The price in which the buyer is willing to buy the property.
* **Actual Price**: The price for which the property is actually bought or sold.
* **Keyword**: A term or phrase used to search for specific contacts or properties. They can match any stored attribute such as name, address, tag, etc.

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<box type="info" seamless>

**Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.
</box>

### Launch and shutdown

1. Initial launch

   1. Ensure you have Java `17` or a newer version installed in your computer.
   If you're not sure, you can download and install it from [this link](https://www.oracle.com/java/technologies/javase-jdk17-downloads.html).

   2. Download the jar file from [this link](https://github.com/AY2425S1-CS2103T-F10-3/tp/releases) and copy into an empty folder

   3. Run the application by typing `java -jar "[CS2103T-F10-3][NUStates].jar"` in a terminal from the folder with `[CS2103T-F10-3][NUStates].jar`. The window size may not be optimum.

2. Saving window preferences

   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   2. Re-launch the app by following `Initial launch` step above.<br>
       Expected: The most recent window size and location is retained.

### Deleting a person

1. Deleting a person while all persons are being shown

   1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

   1. Test case: `delete 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

   1. Test case: `delete 0`<br>
      Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

### Sorting properties for a person

1. Sorting properties for a person

   1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

   1. Test case: `sorti 1 f/Price o/H`<br>
      Expected: Properties - both properties to buy and sell - for the first person are sorted based on property price. Details of the how the properties are sorted shown in the status message.

   1. Test case: `sorti 0 f/Price o/L`<br>
      Expected: No properties are sorted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect sort commands to try: `sorti`, `sorti x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

### Adding a new property to buy or sell for a person

1. Adding a new property to buy or sell for a person

   1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

   1. Test case: `addSell 1 ht/a sp/1200000 pc/431244 un/12-24 t/Near MRT`<br>
      Expected: A new property to buy is added for the first person. Details of the new property shown in the status message.

   1. Test case: `addBuy 0 ht/a sp/1205000 pc/455677 un/1-45`<br>
      Expected: No property is added. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect add property commands to try: `addBuy`, `addSell x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.
