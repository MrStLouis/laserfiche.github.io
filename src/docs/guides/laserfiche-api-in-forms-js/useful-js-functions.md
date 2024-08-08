---
layout: default
title: Useful JavaScript Functions
nav_order: 4
has_children: false
parent: Using the Laserfiche API from Forms JS
grand_parent: Guides
---

<!--© 2024 Laserfiche.
See LICENSE-DOCUMENTATION and LICENSE-CODE in the project root for license information.-->

# Useful JavaScript Functions
- [Working with fields](#working-with-fields)
  - [Finding Fields](#find-fields)
  - [Update Table Rows](#update-table-rows)
  - [Set Table Field Values](#set-table-field-values)
- [API Helpers](#api-helpers)
  - [Required API Helper](#required-api-helper)
  - [Get Repositories](#get-repositories)
  - [Get Metadata](#metadata)
  - [Update Metadata](#update-metadata)
- [HTML Components](#html-components)
  - [Full Field HTML](#full-field-html)
  - [Loading Bar](#loading-bar)
  - [Modal](#modal)

{: .note }
**Hint:** Click on the copy icon in the top right corner of the code block to copy the code to your clipboard.

## Working with fields

### Find Fields
<!-- {% raw %} -->
```javascript
/**
 * Finds a field by id and returns an array of fields. Array is empty if not found.
 * @param {LFFormId} field
 * @returns {LFFormField[]}
 * @preserve
 */
const findField = (field) => {
  if (field.fieldId) {
    return LFForm.findFieldsByFieldId(field.fieldId);
  }
  if (field.variableId) {
    return LFForm.findFieldsByVariableId(field.variableId);
  }
  if (field.variableName) {
    return LFForm.findFieldsByVariableName(field.variableName);
  }
  throw new Error('Field must have a fieldId, variableId, or variableName');
};
/**
 * Finds a field by id or array of ids and returns an array of fields. Array is empty if not found.
 * @param {LFFormId} fieldId
 * @returns {LFFormField[]}
 * @preserve
 */
const findFieldByIdParam = (fieldId) => {
  if (fieldId === void 0 || fieldId === null) {
    return [];
  }
  if (Array.isArray(fieldId)) {
    return fieldId.flatMap((f) => findField(f));
  }
  return findField(fieldId);
};
/**
 * Finds a field by id and returns null if no fields are found.
 * @param {LFFormId} field
 * @returns {LFFormField[]}
 * @preserve
 */
const findFieldOrNull = (field) => {
  const fields = findField(field);
  if (fields.length === 0) {
    return null;
  }
  return fields;
};
```

### Update Table Rows
```javascript
/**
 * Updates the number of rows in a table field dynamically by row count
 * @param {LFFormId} tableFieldId table field to update
 * @param {number} rowCount number of rows to update the table to
 * @returns a promise showing the result of the updateTableRows operation
 * @preserve
 */
const updateTableRows = async (tableFieldId, rowCount) => {
  const currentTableFieldValues = LFForm.getFieldValues(tableFieldId);
  const curRowCount = currentTableFieldValues.length;
  if (curRowCount < rowCount) {
    await LFForm.addRow(tableFieldId, rowCount - curRowCount);
  } else if (curRowCount > rowCount) {
    await LFForm.deleteRow(
      tableFieldId,
      ...Array.from(Array(curRowCount - rowCount)).map(
        (_, i) => curRowCount - i - 1
      )
    );
  }
};
```
### Set Table Field Values
```javascript
/**
 * See https://answers.laserfiche.com/questions/220269/LFForm-Object-setFieldValue-support-for-collections-and-tables#220319 for more information
 * @param {LFFormId} tableFieldId table field to set values on
 * @param {LFFormFieldValueType[][]} values values to set on the table, order of values dictated by options.valueOrder
 * @param {{ 
 *   replaceMode: 'replace' | 'append',
 *   valueOrder: 'row' | 'column'
 * }} options replaceMode to replace or append values, valueOrder to set values by row or column
 * @returns a promise showing the result of the setFieldValues operation
 * @preserve
 */
const setFieldValues = async (tableFieldId, values, options) => {
  const { replaceMode = 'replace', valueOrder = 'row' } = options;
  const tableField = findFieldByIdParam(tableFieldId)[0];
  if (replaceMode === 'replace') {
    await updateTableRows(tableFieldId, values.length);
  } else if (replaceMode === 'append') {
    await LFForm.addRow(tableFieldId, values.length);
  }
  const colNumberToFieldId = {};
  const tableTemplate = Object.values(tableField.repeatableTemplate).reduce(
    (acc, field, i) => {
      colNumberToFieldId[i] = field.fieldId;
      acc[field.fieldId] = [];
      return acc;
    },
    {}
  );
  if (replaceMode === 'append') {
    for (const fieldId of Object.keys(tableTemplate)) {
      tableTemplate[fieldId].push(
        ...LFForm.getFieldValues({ fieldId: Number(fieldId) })
      );
    }
  }
  if (valueOrder === 'row') {
    for (const row of values) {
      for (let col = 0; col < row.length; col++) {
        tableTemplate[colNumberToFieldId[col]].push(row[col]);
      }
    }
  } else if (valueOrder === 'column') {
    for (let col = 0; col < values.length; col++) {
      for (const row of values[col]) {
        tableTemplate[colNumberToFieldId[col]].push(row);
      }
    }
  }
  return Promise.allSettled(
    Object.entries(tableTemplate).map(([fieldId, values2]) => {
      return LFForm.setFieldValues({ fieldId: Number(fieldId) }, values2);
    })
  );
};
```
## API Helpers
### Required API Helper
```javascript
/**
 * Required helper function for all api operations. Do not modify.
 * @param {string | number | LFFormId | LFFormField} entryIdField
 * @returns {number | undefined}
 * @preserve
 */
const resolveEntryIdField = (entryIdField) => {
  let resolvedEntryId = void 0;
  if (typeof entryIdField === 'string' || typeof entryIdField === 'number') {
    resolvedEntryId = entryIdField;
  } else if (entryIdField) {
    resolvedEntryId = LFForm.getFieldValues(findFieldByIdParam(entryIdField));
  } else {
    const resolvedField =
      findFieldOrNull({ variableName: 'Entry_ID' }) ??
      findFieldOrNull({ variableName: 'EntryId' });
    if (resolvedField) {
      resolvedEntryId = LFForm.getFieldValues(resolvedField);
    }
  }
  const entryId = Number(resolvedEntryId);
  if (isNaN(entryId) || entryId === 0) {
    return;
  }
  return entryId;
};
/**
 * Required helper function for all api operations. Do not modify.
 * @param options
 * @returns {{apiClient: RepositoryApiClient, repositoryId: string, entryId: number}}
 * @preserve
 */
const resolveDefaultRepositoryAPIOptions = async (options) => {
  const apiClient =
    options.apiClient ?? (await LFForm.getLaserficheAPIClient('Default'));
  const repositoryId = options.repositoryId || (await getRepositories());
  if (Array.isArray(repositoryId)) {
    throw new Error('Repository ID is required');
  }
  const entryId = resolveEntryIdField(options.entryIdField);
  if (!entryId) {
    throw new Error('Entry ID is required');
  }
  return {
    entryId,
    repositoryId,
    apiClient,
  };
};
```
### Get Repositories
```javascript
/**
 * Returns a list of repositories. Errors if no repositories are found.
 * @param {RepositoryApiClient} apiClient
 * @returns {Promise<Repository[]>}
 * @preserve
 */
const getRepositories = async (apiClient) => {
  const resolvedApiClient =
    apiClient ?? (await LFForm.getLaserficheAPIClient('Default'));
  const repositories =
    await resolvedApiClient.repositoriesClient.listRepositories({});
  if (!repositories.value || repositories.value.length === 0) {
    throw new Error('No repositories found');
  }
  return repositories.value;
};
```
### Metadata
```javascript
/**
 * Sets a form field with generic data. This function is used to set a form field with generic data.
 * It assumes the data is in a valid format, and only converts the structure of the value to meet the field's requirement by type.
 * @param {LFFormField} formField
 * @param {string | { values?: string[] }}value
 * @returns {Promise<boolean>}
 * @preserve
 */
const setGenericFormFieldWithData = async (formField, value) => {
  const genericArray = typeof value === 'object' ? value : { values: [value] };
  if (genericArray.values && !Array.isArray(genericArray.values)) {
    genericArray.values = [genericArray.values];
  }
  if (
    !genericArray ||
    !genericArray.values ||
    genericArray.values.length === 0 ||
    genericArray.values[0] === null
  ) {
    return false;
  }
  const isInCollection =
    formField.settings.isInCollection || formField.settings.isInTable;
  const values = genericArray.values;
  if (formField.componentType === 'Checkbox') {
    await LFForm.setFieldValues(
      formField,
      isInCollection
        ? values.map((value2) => ({ value: [value2] }))
        : { value: [values[0]] }
    );
  } else if (
    formField.componentType === 'Radio' ||
    formField.componentType === 'Dropdown'
  ) {
    await LFForm.setFieldValues(
      formField,
      isInCollection
        ? values.map((value2) => ({ value: value2.toString() }))
        : { value: values[0].toString() }
    );
  } else if (formField.componentType === 'DateTime') {
    await LFForm.setFieldValues(
      formField,
      isInCollection
        ? values.map((value2) => ({ dateStr: value2, timeStr: '' }))
        : { dateStr: values[0], timeStr: '' }
    );
  } else {
    await LFForm.setFieldValues(
      formField,
      isInCollection ? values : values.join(', ')
    );
  }
  return true;
};
/**
 * Checks if a field id is equal to a field. Can also check equivalence of two fields.
 * @param {LFFormId} id
 * @param {LFFormField} field
 * @returns
 * @preserve
 */
const isFieldIdEqualField = (id, field) => {
  if (id.fieldId) return id.fieldId === field.fieldId;
  if (id.variableId) return id.variableId === field.settings.attributeId;
  if (id.variableName) return id.variableName === field.settings.attributeName;
  return false;
};
/**
 * Maps an entry's metadata to a form by matching fields based on the field's label or a custom field id.
 * Optionally disables fields after mapping.
 * @param {{
 *  entryIdField: string | number | LFFormId | LFFormField,
 *  repositoryId: string,
 *  apiClient: RepositoryApiClient,
 *  options: {
 *    makeFieldsDisabled: boolean | LFFormId[],
 *    matchFieldBy: 'label' | Record<string, LFFormId>,
 * }}} options
 * @returns
 * @preserve
 */
const mapEntryToForm = async ({ options, ...apiOptions }) => {
  const { entryId, repositoryId, apiClient } =
    await resolveDefaultRepositoryAPIOptions(apiOptions);
  const entryInfo = await apiClient.entriesClient.getEntry({
    repositoryId,
    entryId,
    select: 'name',
  });
  const fields = await apiClient.entriesClient.listFields({
    repositoryId,
    entryId,
    formatFieldValues: true,
  });
  const metadataFieldMap =
    fields.value?.reduce((acc, field) => {
      if (!field.name) return acc;
      acc[field.name] = field;
      return acc;
    }, {}) ?? {};
  metadataFieldMap['Entry Name'] = { values: [entryInfo.name ?? ''] };
  const { makeFieldsDisabled = false, matchFieldBy = 'label' } = options;
  const mappedFields = [];
  if (typeof matchFieldBy === 'string') {
    LFForm.findFields((formField) => {
      const metadataField = metadataFieldMap[formField.settings[matchFieldBy]];
      if (!metadataField) return false;
      mappedFields.push(
        setGenericFormFieldWithData(formField, metadataField).then((result) =>
          result ? formField : null
        )
      );
      return false;
    });
  } else if (typeof matchFieldBy === 'object') {
    Object.entries(matchFieldBy).map(([key, value]) => {
      const formField = findFieldOrNull(value);
      if (!formField) {
        return;
      }
      const metadataField = metadataFieldMap[key];
      mappedFields.push(
        setGenericFormFieldWithData(formField[0], metadataField).then(
          (result) => (result ? formField[0] : null)
        )
      );
    });
  }
  const setFields = await Promise.allSettled(mappedFields);
  if (makeFieldsDisabled === false) return setFields;
  await Promise.allSettled(
    setFields.map((result) => {
      if (result.status === 'rejected') return null;
      const field = result.value;
      if (
        field &&
        (makeFieldsDisabled === true ||
          makeFieldsDisabled.findIndex((id) => isFieldIdEqualField(id, field)) >
            -1)
      ) {
        return LFForm.disableFields(field);
      }
      return null;
    })
  );
  return setFields.filter((result) => result.status === 'fulfilled');
};

/**
 * Updates the number of rows in a table field dynamically by row count
 * @param {LFFormId} tableFieldId table field to update
 * @param {number} rowCount number of rows to update the table to
 * @returns a promise showing the result of the updateTableRows operation
 * @preserve
 */
const updateTableRows = async (tableFieldId, rowCount) => {
  const currentTableFieldValues = LFForm.getFieldValues(tableFieldId);
  const curRowCount = currentTableFieldValues.length;
  if (curRowCount < rowCount) {
    await LFForm.addRow(tableFieldId, rowCount - curRowCount);
  } else if (curRowCount > rowCount) {
    await LFForm.deleteRow(
      tableFieldId,
      ...Array.from(Array(curRowCount - rowCount)).map(
        (_, i) => curRowCount - i - 1
      )
    );
  }
};
/**
 * @param {LFFormId} tableFieldId table field to set values on
 * @param {LFFormFieldValueType[][]} values values to set on the table, order of values dictated by options.valueOrder
 * @param {SetFieldValueOptions} options replaceMode to replace or append values, valueOrder to set values by row or column
 * @returns a promise showing the result of the setFieldValues operation
 * @preserve
 */
const setFieldValues = async (tableFieldId, values, options) => {
  const { replaceMode = 'replace', valueOrder = 'row' } = options;
  const tableField = findFieldByIdParam(tableFieldId)[0];
  if (replaceMode === 'replace') {
    await updateTableRows(tableFieldId, values.length);
  } else if (replaceMode === 'append') {
    await LFForm.addRow(tableFieldId, values.length);
  }
  const colNumberToFieldId = {};
  const tableTemplate = Object.values(tableField.repeatableTemplate).reduce(
    (acc, field, i) => {
      colNumberToFieldId[i] = field.fieldId;
      acc[field.fieldId] = [];
      return acc;
    },
    {}
  );
  if (replaceMode === 'append') {
    for (const fieldId of Object.keys(tableTemplate)) {
      tableTemplate[fieldId].push(
        ...LFForm.getFieldValues({ fieldId: Number(fieldId) })
      );
    }
  }
  if (valueOrder === 'row') {
    for (const row of values) {
      for (let col = 0; col < row.length; col++) {
        tableTemplate[colNumberToFieldId[col]].push(row[col]);
      }
    }
  } else if (valueOrder === 'column') {
    for (let col = 0; col < values.length; col++) {
      for (const row of values[col]) {
        tableTemplate[colNumberToFieldId[col]].push(row);
      }
    }
  }
  return Promise.allSettled(
    Object.entries(tableTemplate).map(([fieldId, values2]) => {
      return LFForm.setFieldValues({ fieldId: Number(fieldId) }, values2);
    })
  );
};
```
### Update Metadata
```javascript
/**
 * Get any field's value as a string or string array.
 * Helpful to automatically send form data to the repository regardless of the field's type.
 * @param field field to retrieve value from
 * @param isMultiValue whether the field is a multi-value field to
 * @returns string if isMultiValue is false, string[] if isMultiValue is true
 * @preserve
 */
function getFieldValueAsString(field, isMultiValue) {
  const fieldValue = LFForm.getFieldValues(field);
  const valueAsList = Array.isArray(fieldValue) ? fieldValue : [fieldValue];
  const output = valueAsList.map((value) => {
    if (typeof value === 'string') return value;
    if (typeof value === 'number') return value.toString();
    if (field.componentType === 'DateTime') {
      const dateVal = value;
      return new Date(dateVal.dateTimeObj).toISOString();
    }
    if (field.componentType === 'Checkbox') {
      const checkboxVal = value;
      return checkboxVal.otherChoiceValue
        ? [checkboxVal.otherChoiceValue]
            .concat(checkboxVal.value?.map((v) => v.toString()) ?? [])
            .join(', ')
        : checkboxVal.value?.map((v) => v.toString()).join(', ');
    }
    if (field.componentType === 'Radio') {
      const singleValue = value;
      return singleValue.otherChoiceValue ?? singleValue.value.toString();
    }
    if (field.componentType === 'Address') {
      const address = value;
      return Object.values(address)
        .filter((v) => v && v !== '')
        .join(', ');
    }
    return value.toString();
  });
  return isMultiValue ? output : output.join('; ');
}
/**
 * Patch entry metadata with new metadata values. Merges new metadata with existing fields when saving.
 * @param {{
 *  entryIdField: string | number | LFFormId | LFFormField,
 *  repositoryId: string,
 *  apiClient: RepositoryApiClient,
 *  options: {
 *    newMetadata: LFFormField[],
 * }}} options
 * @returns {Promise<FieldCollectionResponse | undefined>}
 * @preserve
 */
const patchEntryMetadata = async ({ options, ...apiOptions }) => {
  if (options.newMetadata.length === 0) return;
  const { apiClient, repositoryId, entryId } =
    await resolveDefaultRepositoryAPIOptions(apiOptions);
  const metadataFields = await apiClient.entriesClient.listFields({
    repositoryId,
    entryId,
    formatFieldValues: false,
  });
  const metadataMap = metadataFields.value?.reduce((acc, field) => {
    if (!field.name) return acc;
    acc[field.name] = {
      name: field.name,
      values: field.values,
      isMultiValue: field.isMultiValue,
    };
    return acc;
  }, {});
  if (!metadataMap) return;
  for (const formField of options.newMetadata) {
    const metadataField = metadataMap[formField.settings.label];
    if (!metadataField) continue;
    const value = LFForm.getFieldValues(formField);
    if (!value) continue;
    if (metadataField.isMultiValue) {
      metadataField.values = getFieldValueAsString(formField, true);
    } else {
      const newValue = getFieldValueAsString(
        formField,
        metadataField.isMultiValue ?? false
      );
      metadataField.values = [newValue];
    }
  }
  const request = {
    fields: Object.values(metadataMap),
  };
  const patchResponse = apiClient.entriesClient.setFields({
    repositoryId,
    entryId,
    request,
  });
  return patchResponse;
};
```


### Search
```javascript
const defaultSearchOptions = {
  pollInterval: 250,
  maxPollAttempts: 20,
  maxPageSize: 1e3,
};
/**
 * @param {RepositoryApiClient} apiClient The repository api client
 * @param {string} repositoryId The id of the repository to search
 * @param {string} searchCommand The search command to run
 * @param {SearchOptions} searchOptions Optional search options
 * @returns {Promise<EntryCollectionResponse | null>} The search results or null if the search was canceled
 * @preserve
 */
const searchAsync = async (
  apiClient,
  repositoryId,
  searchCommand,
  searchOptions
) => {
  const { pollInterval, fuzzyFactor, fuzzyType, maxPageSize, maxPollAttempts } =
    {
      ...defaultSearchOptions,
      ...searchOptions,
    };
  const openTasks = await apiClient.tasksClient.listTasks({ repositoryId });
  const openTaskIds = openTasks.value?.map((task) => task.id) ?? [];
  if (openTaskIds?.length && openTaskIds.length > 1) {
    console.warn('Cancelling previous search task');
    await apiClient.tasksClient
      .cancelTasks({
        repositoryId,
        taskIds: openTaskIds.filter((t) => t !== void 0).slice(1),
      })
      .catch();
  }
  const startTask = await apiClient.searchesClient.startSearchEntry({
    repositoryId,
    request: {
      searchCommand,
      fuzzyFactor,
      fuzzyType,
    },
  });
  const { taskId } = startTask;
  if (taskId === void 0) return null;
  searchOptions?.onSearchStart?.(startTask);
  const finalTaskResponse = await new Promise((resolve, reject) => {
    let awaitingInterval = false;
    let currentPollCount = 0;
    const timeout = setInterval(async () => {
      console.log(currentPollCount);
      if (awaitingInterval) return;
      if (currentPollCount >= maxPollAttempts) {
        clearInterval(timeout);
        reject('Search timed out');
        return;
      }
      currentPollCount++;
      awaitingInterval = true;
      const taskResponse = await apiClient.tasksClient.listTasks({
        repositoryId,
        taskIds: [taskId],
      });
      searchOptions?.onPollIteration?.(taskResponse.value?.[0]);
      if (taskResponse?.value?.[0].status === 'Completed') {
        clearInterval(timeout);
        resolve(taskResponse.value[0].result);
        return;
      }
      if (taskResponse?.value?.[0].status === 'Failed') {
        clearInterval(timeout);
        reject(
          new Error(
            taskResponse.value[0].errors?.[0]?.title ?? 'Unknown search error'
          )
        );
        return;
      }
      if (taskResponse?.value?.[0].status === 'Cancelled') {
        clearInterval(timeout);
        resolve(void 0);
        return;
      }
    }, pollInterval);
  });
  if (!finalTaskResponse) {
    return null;
  }
  const searchResponse = await apiClient.searchesClient.listSearchResults({
    repositoryId,
    taskId,
    prefer: `odata.maxpagesize=${maxPageSize}`,
  });
  return searchResponse;
};
/**
 * @param {LFFormId} tableFieldId
 * @param {EntryCollectionResponse} results
 * @param {{ [fieldId: number]: (entry: Entry) => LFFormSetFieldValueType }} valueMap
 * @returns {Promise<PromiseSettledResult<LFFormPromiseResponse>[] | undefined>}
 * @preserve
 */
const fillTableWithSearchResults = async (tableFieldId, results, valueMap) => {
  if (!results?.value?.length) return;
  const fieldValues = Object.keys(valueMap).reduce((acc, fieldId) => {
    acc[fieldId] = [];
    return acc;
  }, {});
  for (const result of results.value) {
    for (const fieldId of Object.keys(valueMap)) {
      fieldValues[fieldId].push(valueMap[Number(fieldId)](result));
    }
  }
  await updateTableRows(tableFieldId, results.value.length);
  return Promise.allSettled(
    Object.entries(fieldValues).map(([fieldId, values]) =>
      LFForm.setFieldValues({ fieldId: Number(fieldId) }, values)
    )
  );
};
```

## HTML Components

### Full Field HTML

```javascript
/**
 * Automatically sets a full field HTML content above or below the field.
 * Passes the content embedded in a div with absolute positioning.
 * @param {LFFormId | LFFormField} formField
 * @param {'textAbove' | 'textBelow'} placement
 * @param {string} content
 * @param {string} styles
 * @returns {Promise<void>}
 * @preserve
 */
const fullFieldHtml = async (formField, placement, content, styles) => {
  const html = content
    ? /* html */
      `
  <div
    style="position: absolute;
           top: 0;
           left: 0;
           display: flex;
           align-items: center;
           justify-content: center;
           width: 100%;
           z-index: 1;
           height: 100%;${styles}"
  >
    ${content} 
  </div>
`
    : '';
  await LFForm.changeFieldSettings(formField, {
    [placement]: html,
  });
};
```

### Loading Bar

```javascript
function makeLoadingBar(curPrecent, loadingBarOptions) {
  const { height = 200, type, text } = loadingBarOptions || {};
  return `
<div class="container container-flex" style="display: flex;height: ${height}px;flex-direction: column;justify-content: center; width:100%">
  ${text ? `<h6 style="text-align: center;">${text}</h6>` : ''}
  <div class="progress">
    <div class="progress-bar progress-bar-striped progress-bar-animated ${
      type ? `bg-${type}` : ''
    }" role="progressbar" aria-valuenow="${curPrecent}" aria-valuemin="0" aria-valuemax="100" style="width: ${curPrecent}%"></div>
  </div>
</div>`;
}
```

Full Field Loading Bar

```javascript
await fullFieldHtml(
  formField,
  'textAbove',
  makeLoadingBar(0, { height: 20, type: 'info', text: 'Loading...' }),
  'background-color: white;'
);
```

### Modal

````javascript
class LFFormModal {
  customHtmlField;
  /**
   * @param {LFFormId} customHtmlFieldId
   * @param {{
   *  size: 'sm' | 'default' | 'lg' | 'xl',
   *  showBackdrop: boolean,
   *  allowBackdropDismiss: boolean
   * }} modalOptions
   * @example
   * ```javascript
   * const modal = new LFFormModal(formFields.modalField, {
   *  size: 'default',
   *  showBackdrop: true,
   *  allowBackdropDismiss: false,
   * });
   * // Set details without showing modal
   * modal.setDetails({
   *   title: 'Modal Title',
   *   content: 'This is the modal content',
   *   buttons: [{
   *    label: 'Close',
   *    key: 'close',
   *    style: 'default'
   *   }, {
   *    label: 'OK',
   *    key: 'ok',
   *    style: 'primary'
   *  }]
   * }, false);
   * // Set close handlers by button key (not label)
   * modal.onClose('ok', () => {
   *  console.log('OK button clicked');
   * });
   * // Show modal
   * await modal.show();
   * ```
   * @preserve
   */
  constructor(customHtmlFieldId, modalOptions) {
    this.customHtmlField = findField(customHtmlFieldId)?.[0];
    if (!this.customHtmlField) {
      throw new Error('Custom HTML Field not found');
    }
    this.setOptions(modalOptions ?? {}, false);
    window.dismissModal = this.#dismissModal;
    void this.hide();
  }
  show = async () => {
    await this.#setModalHtml();
    this.#modalDismissed = false;
    return LFForm.removeCSSClasses(this.customHtmlField, 'invisible');
  };
  hide = () => {
    return LFForm.addCSSClasses(this.customHtmlField, 'invisible');
  };
  setDetails({ title, content, buttons, modalOptions }, rerender = true) {
    if (title !== void 0) this.setTitleDetail(title, false);
    if (content !== void 0) this.setContentDetails(content, false);
    if (buttons !== void 0) this.setButtonDetails(buttons, false);
    if (modalOptions !== void 0) this.setOptions(modalOptions, false);
    if (rerender === true) return this.#setModalHtml();
    return void 0;
  }
  setTitleDetail(title, rerender = true) {
    this.#title = title;
    if (rerender) return this.#setModalHtml();
    return;
  }
  setButtonDetails(buttons, rerender = true) {
    this.#buttons = buttons;
    if (rerender) return this.#setModalHtml();
    return;
  }
  #closeHandlers = {};
  onClose(button, callback) {
    const handlers = this.#closeHandlers[button] || [];
    handlers.push(callback);
    this.#closeHandlers[button] = handlers;
  }
  resetCloseHandlers(button) {
    if (button) {
      this.#closeHandlers[button] = [];
    } else {
      this.#closeHandlers = {};
    }
  }
  setContentDetails(content, rerender = true) {
    this.#content = content;
    if (rerender) return this.#setModalHtml();
    return;
  }
  setOptions(modalOptions, rerender = true) {
    this.#modalOptions = {
      ...this.#defaultModalOptions,
      ...this.#modalOptions,
      ...modalOptions,
    };
    if (rerender) return this.#setModalHtml();
    return;
  }
  #modalOptions = {};
  #defaultModalOptions = {
    size: 'default',
    allowBackdropDismiss: true,
  };
  #title = 'Modal Title';
  #content = 'This is the modal content';
  #buttons = [];
  #defaultButtons = [
    {
      label: 'Close',
      key: 'close',
      style: 'secondary',
    },
  ];
  #modalDismissed = false;
  #blockClose = false;
  #dismissModal = async (button) => {
    console.log(button);
    if (this.#modalDismissed || this.#blockClose) return;
    this.#blockClose = false;
    this.#modalDismissed = true;
    const handlers = this.#closeHandlers[button] || [];
    for (const handler of handlers) {
      await handler();
    }
    return this.hide();
  };
  #generateModalHtml = () => {
    const useButtons =
      this.#buttons.length === 0 ? this.#defaultButtons : this.#buttons;
    const allowBackdropDismiss = this.#modalOptions.allowBackdropDismiss;
    const modalSize =
      this.#modalOptions.size !== 'default'
        ? `modal-${this.#modalOptions.size}`
        : '';
    const showBackdrop = this.#modalOptions.showBackdrop ?? false;
    return `
    <div>
  <div class="modal fade show d-block" tabindex="-1" ${
    allowBackdropDismiss ? 'onclick="window.dismissModal(`close`)"' : ''
  }>
    <div class="modal-dialog modal-dialog-centered ${modalSize}" ${
      allowBackdropDismiss ? 'onclick="window.dismissModal(`block-close`)"' : ''
    }>
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">${this.#title}</h5>
          <button type="button" class="close" data-dismiss="modal" aria-label="Close"
            onclick="window.dismissModal('close')">
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body">
          ${this.#content}
        </div>
        <div class="modal-footer">
          ${useButtons
            .map(({ key, style, label }) => {
              return `<button type="button" class="btn btn-${style}"
            onclick="window.dismissModal('${key}')">${label}</button>`;
            })
            .join('')}
        </div>
      </div>
    </div>
  </div>
  <div class="d-block" tabindex="-1" ${
    showBackdrop
      ? 'style="background-color: rgba(0, 0, 0, .2); backdrop-filter: blur(5px); position: fixed; top: 0; left: 0; right: 0; bottom: 0; z-index: 1049;"'
      : ''
  } ${allowBackdropDismiss ? 'onclick="window.dismissModal(`close`)"' : ''}>
  </div>
</div>`;
  };
  #setModalHtml = () => {
    const modalHtml = this.#generateModalHtml();
    return LFForm.changeFieldSettings(this.customHtmlField, {
      content: modalHtml,
    });
  };
}
````

Modal Example

```javascript
const modal = new LFFormModal(formFields.modalField, {
  size: 'default',
  showBackdrop: true,
  allowBackdropDismiss: false,
});
// Set details without showing modal by passing false as the second argument
modal.setDetails({
  title: 'Modal Title',
  content: 'This is the modal content',
  buttons: [{
    label: 'Close',
    key: 'close',
    style: 'default'
  }, {
    label: 'OK',
    key: 'ok',
    style: 'primary'
  }]
}, false);
// Set close handlers by button key (not label)
modal.onClose('ok', () => {
  console.log('OK button clicked');
});
// Show modal
await modal.show();
```
<!-- {% endraw %} -->
