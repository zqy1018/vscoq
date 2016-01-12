#CoqTop XML Protocol#

* [Commands](#commands)
  - [Add](#command-add)
  - [EditAt](#command-editAt)
  - [Init](#command-init)
  - [Goal](#command-goal)
* [Feedback messages](#feedback)

Sentences: each command sent to CoqTop is a "sentence"; they are typically terminated by ".\s" (followed by whitespace).
Examples: "Lemma a: True.", "(* asdf *) Qed.", "auto; reflexivity."
In practice, the command sentences sent to CoqTop are terminated at the "." and start with any previous whitespace.
Each sentence is assigned a unique stateId after being sent to Coq (via Add).
States:
  * Processing: has been received by Coq and has no obvious syntax error (that would prevent future parsing)
  * Processed:
  * InProgress:
  * Incomplete: the validity of the sentence cannot be checked due to a prior error
  * Complete:

Sentences & Focusing.
Focus: the stack of sentences representing the current proof being worked on.
Context: the sentences that occur before and after the current focus.
Focusing: take a range of sentences and make then the current focus; place all other sentences in the context. Assumes context started empty.
Unfocusing: append the context to the top and bottom of the current stack (context becomes empty).
Push: push on top of current stack
Pop: pop from current stack

  

--------------------------

## <a name="commands">
</a>Commands

### <a name="command-add">
</a>**Add(stateId: integer, command: string)**
```html
<call val="Add">
  <pair>
    <pair>
      <string>${command}</string>
      <int>${editId}</int>
    </pair>
    <pair>
      <state_id val="${stateId}"/>
      <bool val="false"/>
    </pair>
  </pair>
</call>```
### *Returns*
* The added command is given a fresh `stateId` and becomes the next "tip".
```html
<value val="good">
    <pair>
      <state_id val="${newStateId}"/>
      <pair>
        <union val="in_l">
<unit/>
</union>
        <string>${message}</string>
      </pair>
    </pair>
</value>
```
* When closing a focused proof (in the middle of a bunch of interpreted commands),
the `Qed` will be assigned a prior `stateId` and `nextStateId` will be the id of an already-interpreted
state that should become the next tip. 
```html
<value val="good">
    <pair>
      <state_id val="${stateId}"/>
      <pair>
        <union val="in_r">
<state_id val="${nextStateId}"/>
</union>
        <string>${message}</string>
      </pair>
    </pair>
</value>
```
* Failure:
  - Syntax error. Error offsets are with respect to the start of the sentence.
  ```html
  <value val="fail"
        loc_s="${startOffsetOfError}"
        loc_e="${endOffsetOfError}">
      <state_id val="${stateId}"/>
      ${errorMessage}
  </value>
  ```
  - Another error (e.g. Qed before goal complete)
  ```html
  <value val="fail">
<state_id val="${stateId}"/>${errorMessage}</value>
  ```


### <a name="command-editAt">
</a>**EditAt(stateId: integer)**
### *Returns*
* Simple backtrack; focused stateId becomes the parent state
```html
<value val="good">
    <union val="in_l">
<unit/>
</union>
</value>```
* New focus; focusedQedStateId is the closing Qed of the new focus; senteneces between the two should be cleared
```html
<value val="good">
    <union val="in_r">
      <pair>
        <state_id val="${focusedStateId}"/>
        <pair>
          <state_id val="${focusedQedStateId}"/>
          <state_id val="${oldFocusedStateId}"/>
        </pair>
      </pair>
    </union>
</value>```
* Failure: If `stateId` is in an error-state and cannot be jumped to, `errorFreeStateId` is the parent state of ``stateId` that shopuld be edited instead. 
```html
<value val="fail" loc_s="${startOffsetOfError}" loc_e="${endOffsetOfError}">
    <state_id val="${errorFreeStateId}"/>
    ${errorMessage}
</value>
```


### <a name="command-init">
</a>**Init()**
* No options.
```html
<call val="Init">
<option val="none"/>
</call>```
* With options.
```html
<call val="Init">
<option val="some">
<string>${options}</string>
</option>
</call>```

### *Returns*
* The initial stateId (not associated with a sentence)
```html
<value val="good">
<state_id val="${initialStateId}"/>
</value>```



### <a name="command-goal">
</a>**Goal()**
```html
<call val="Goal">
<unit/>
</call>
```
### *Returns*
* If there is a goal. `backgroundGoals`, `shelvedGoals`, and `abandonedGoals` have the same structure as the first set of (current/foreground) goals. 
```html
<value val="good">
  <option val="some">
  <goals>
    <list>
      <goal>
        <string>3</string>
        <list>
          <string>${hyp1}</string>
          ...
          <string>${hypN}</string>
        </list>
        <string>${goal}</string>
      </goal>
      ...
      ${goalN}
    </list>
    ${backgroundGoals}
    ${shelvedGoals}
    ${abandonedGoals}
  </goals>
  </option>
</value>
```
* No goal:
```html
<value val="good">
<option val="none"/>
</value>
```



### <a name="command-status">
</a>**Status(force)**
CoqIDE typically sets `force` to `false`. 
```html
<call val="Status">
<bool val="${force}"/>
</call>
```
### *Returns*
*  
```html
<status>
    <string>${path}</string>
    <string>${proofName}</string>
    <string>${allProofs}</string>
    <string>${proofNumber}</string>
</status>
```



### <a name="command-query">
</a>**Query(query, stateId)**
```html
<call val="Query">
<string>${query}</string>
<state_id val="${stateId}"/>
</call>
```
### *Returns*
*
```html
<TODO/>
```



### <a name="command-evars">
</a>**Evars()**
```html
<call val="Evars">
<unit/>
</call>
```
### *Returns*
*
```html
<TODO/>
```



### <a name="command-hints">
</a>**Hints()**
```html
<call val="Hints">
<unit/>
</call>
```
### *Returns*
*
```html
<TODO/>
```



### <a name="command-search">
</a>**Search(...)**
```html
<call val="Search">...</call>
```
### *Returns*
*
```html
<TODO/>
```



### <a name="command-getoptions">
</a>**GetOptions()**
```html
<call val="GetOptions">
<unit/>
</call>
```
### *Returns*
*
```html
<TODO/>
```



### <a name="command-setoptions">
</a>**SetOptions(options)**
Sends a list of option settings, where each setting roughly looks like:
`([optionNamePart1, ..., optionNamePartN], value)`.
```html
<call val="SetOptions">
    <list>
      <pair>
        <list>
          <string>optionNamePart1</string>
          ...
          <string>optionNamePartN</string>
        </list>
        <option_value val="${typeOfOption}">
          <option val="some">
            ${value}
          </option>
        </option_value>
      </pair>
      ...
      <!-- Example: -->
      <pair>
        <list>
          <string>Printing</string>
          <string>Width</string>
        </list>
        <option_value val="intvalue">
          <option val="some"><int>60</int></option>
        </option_value>
      </pair>
    </list>
</call>
```
CoqIDE sends the following settings (defaults in parentheses):
```
Printing Width : (<option_value val="intvalue"><int>60</int></option_value>),
Printing Coercions : (<option_value val="boolvalue"><bool val="false"/></option_value>),
Printing Matching : (...true...)
Printing Notations : (...true...)
Printing Existential Instances : (...false...)
Printing Implicit : (...false...)
Printing All : (...false...)
Printing Universes : (...false...)
```
### *Returns*
*
```html
<TODO/>
```



### <a name="command-mkcases">
</a>**MkCases(...)**
```html
<call val="MkCases">
<string>...</string>
</call>
```
### *Returns*
*
```html
<TODO/>
```



### <a name="command-stopworker">
</a>**StopWorker(worker)**
```html
<call val="StopWorker">
<string>${worker}</string>
</call>
```
### *Returns*
*
```html
<TODO/>
```



### <a name="command-printast">
</a>**PrintAst()**
```html
<call val="PrintAst">
<unit/>
</call>
```
### *Returns*
*
```html
<TODO/>
```



### <a name="command-annotate">
</a>**Annotate(annotation)**
```html
<call val="Annotate">
<string>${annotation}</string>
</call>
```
### *Returns*
*
```html
<TODO/>
```

-------------------------------

## <a name="feedback">
</a>Feedback messages

Feedback messages are issued out-of-band,
  giving updates on the current state of sentences/stateIds,
  worker-thread status, etc.

* Added Axiom: in response to `Axiom`, `admit`, `Admitted`, etc.
```html
<feedback object="state" route="0">
    <state_id val="${stateId}"/>
    <feedback_content val="addedaxiom" />
</feedback>
```
* Processing
```html
<feedback object="state" route="0">
    <state_id val="${stateId}"/>
    <feedback_content val="processingin">
      <string>${workerName}</string>
    </feedback_content>
</feedback>
```
* Processed
```html
</feedback>
    <feedback object="state" route="0">
      <state_id val="${stateId}"/>
    <feedback_content val="processed"/>
</feedback>
```
* Incomplete
```html
<feedback object="state" route="0">
    <state_id val="${stateId}"/>
    <feedback_content val="incomplete" />
</feedback>
```
* Complete
* GlobRef
* Error. Issued, for example, when a processed tactic has failed or is unknown.
The error offsets may both be 0 if there is no particular syntax involved.
```html
<feedback object="state" route="0">
  <state_id val="${stateId}"/>
  <feedback_content val="errormsg">
    <loc start="${sentenceOffsetBegin}" stop="${sentenceOffsetEnd}"/>
    <string>${errorMessage}</string>
  </feedback_content>
</feedback>
```
* InProgress
```html
<feedback object="state" route="0">
    <state_id val="${stateId}"/>
    <feedback_content val="inprogress">
      <int>1</int>
    </feedback_content>
</feedback>
```
* WorkerStatus
Ex: `workername = "proofworker:0"`
Ex: `status = "Idle"` or `status = "proof: myLemmaName"` or `status = "Dead"`
```html
<feedback object="state" route="0">
    <state_id val="${stateId}"/>
    <feedback_content val="workerstatus">
      <pair>
        <string>${workerName}</string>
        <string>${status}</string>
      </pair>
    </feedback_content>
</feedback>
```
* File Dependencies. Typically in response to a `Require`. Dependencies are *.vo files.
  - State `stateId` directly depends on `dependency`:
  ```html
  <feedback object="state" route="0">
      <state_id val="${stateId}"/>
      <feedback_content val="filedependency">
        <option val="none"/>
        <string>${dependency}</string>
      </feedback_content>
  </feedback>
  ```
  - State `stateId` depends on `dependency` via dependency `sourceDependency`
  ```html
  <feedback object="state" route="0">
      <state_id val="${stateId}"/>
      <feedback_content val="filedependency">
        <option val="some">
<string>${sourceDependency}</string>
</option>
        <string>${dependency}</string>
      </feedback_content>
  </feedback>
  ```
* File loaded. For state `stateId`, module `module` is being loaded from `voFileName`
```html
<feedback object="state" route="0">
  <state_id val="${stateId}"/>
  <feedback_content val="fileloaded">
    <string>${module}</string>
    <string>${voFileName`}</string>
  </feedback_content>
</feedback>
```

