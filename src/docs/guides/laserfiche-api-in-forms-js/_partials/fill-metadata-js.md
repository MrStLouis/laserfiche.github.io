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
  throw new Error("Field must have a fieldId, variableId, or variableName");
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

/**
 * Returns a list of repositories. Errors if no repositories are found.
 * @param {RepositoryApiClient} apiClient 
 * @returns {Promise<Repository[]>}
 * @preserve
 */
const getRepositories = async (apiClient) => {
  const resolvedApiClient = await LFForm.getLaserficheAPIClient("Default");
  const repositories = await resolvedApiClient.repositoriesClient.listRepositories({});
  if (!repositories.value || repositories.value.length === 0) {
    throw new Error("No repositories found");
  }
  return repositories.value;
};

const resolveEntryIdField = (entryIdField) => {
  let resolvedEntryId = void 0;
  if (typeof entryIdField === "string" || typeof entryIdField === "number") {
    resolvedEntryId = entryIdField;
  } else if (entryIdField) {
    resolvedEntryId = LFForm.getFieldValues(
      findFieldByIdParam(entryIdField)
    );
  } else {
    const resolvedField = findFieldOrNull({ variableName: "Entry_ID" }) ?? findFieldOrNull({ variableName: "EntryId" });
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
const resolveDefaultRepositoryAPIOptions = async (options) => {
  const apiClient = options.apiClient ?? await LFForm.getLaserficheAPIClient("Default");
  const repositoryId = options.repositoryId || await getRepositories();
  if (Array.isArray(repositoryId)) {
    throw new Error("Repository ID is required");
  }
  const entryId = resolveEntryIdField(options.entryIdField);
  if (!entryId) {
    throw new Error("Entry ID is required");
  }
  return {
    entryId,
    repositoryId,
    apiClient
  };
};

/**
 * Sets a form field with generic data. This function is used to set a form field with generic data.
 * It assumes the data is in a valid format, and only converts the structure of the value to meet the field's requirement by type.
 * @param {LFFormField} formField
 * @param {string | { values?: string[] }}value 
 * @returns {Promise<boolean>}
 * @preserve
 */
const setGenericFormFieldWithData = async (formField, value) => {
  const genericArray = typeof value === "object" ? value : { values: [value] };
  if (genericArray.values && !Array.isArray(genericArray.values)) {
    genericArray.values = [genericArray.values];
  }
  if (!genericArray || !genericArray.values || genericArray.values.length === 0 || genericArray.values[0] === null) {
    return false;
  }
  const isInCollection = formField.settings.isInCollection || formField.settings.isInTable;
  const values = genericArray.values;
  if (formField.componentType === "Checkbox") {
    await LFForm.setFieldValues(
      formField,
      isInCollection ? values.map((value2) => ({ value: [value2] })) : { value: [values[0]] }
    );
  } else if (formField.componentType === "Radio" || formField.componentType === "Dropdown") {
    await LFForm.setFieldValues(
      formField,
      isInCollection ? values.map((value2) => ({ value: value2.toString() })) : { value: values[0].toString() }
    );
  } else if (formField.componentType === "DateTime") {
    await LFForm.setFieldValues(
      formField,
      isInCollection ? values.map((value2) => ({ dateStr: value2, timeStr: "" })) : { dateStr: values[0], timeStr: "" }
    );
  } else {
    await LFForm.setFieldValues(
      formField,
      isInCollection ? values : values.join(", ")
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
const mapEntryToForm = async ({
  options,
  ...apiOptions
}) => {
  const { entryId, repositoryId, apiClient } = await resolveDefaultRepositoryAPIOptions(apiOptions);
  const entryInfo = await apiClient.entriesClient.getEntry({
    repositoryId,
    entryId,
    select: "name"
  });
  const fields = await apiClient.entriesClient.listFields({
    repositoryId,
    entryId,
    formatFieldValues: true
  });
  const metadataFieldMap = fields.value?.reduce((acc, field) => {
    if (!field.name) return acc;
    acc[field.name] = field;
    return acc;
  }, {}) ?? {};
  metadataFieldMap["Entry Name"] = { values: [entryInfo.name ?? ""] };
  const { makeFieldsDisabled = false, matchFieldBy = "label" } = options;
  const mappedFields = [];
  if (typeof matchFieldBy === "string") {
    LFForm.findFields((formField) => {
      const metadataField = metadataFieldMap[formField.settings[matchFieldBy]];
      mappedFields.push(
        setGenericFormFieldWithData(formField, metadataField).then(
          (result) => result ? formField : null
        )
      );
      return false;
    });
  } else if (typeof matchFieldBy === "object") {
    Object.entries(matchFieldBy).map(([key, value]) => {
      const formField = findFieldOrNull(value);
      if (!formField) {
        return;
      }
      const metadataField = metadataFieldMap[key];
      mappedFields.push(
        setGenericFormFieldWithData(formField[0], metadataField).then(
          (result) => result ? formField[0] : null
        )
      );
    });
  }
  const setFields = await Promise.allSettled(mappedFields);
  if (makeFieldsDisabled === false) return setFields;
  await Promise.allSettled(
    setFields.map((result) => {
      if (result.status === "rejected") return null;
      const field = result.value;
      if (field && (makeFieldsDisabled === true || makeFieldsDisabled.findIndex((id) => isFieldIdEqualField(id, field)) > -1)) {
        return LFForm.disableFields(field);
      }
      return null;
    })
  );
  return setFields.filter((result) => result.status === "fulfilled");
};

// Start of custom code
const main = async () => {
  const apiClient = await LFForm.getLaserficheAPIClient('Default').catch(
    (err) => console.error(err)
  );
  const repositoryId = 'r-MY_REPO_ID_HERE';

  const mappedFields = await mapEntryToForm({
  repositoryId, // From variable above
  entryIdField: "123456", // Can be a field, a string, or a number
  apiClient, // From variable above
  options: { makeFieldsDisabled: [{ fieldId: 2 }] }
});
console.log(mappedFields);
};
void main();
```