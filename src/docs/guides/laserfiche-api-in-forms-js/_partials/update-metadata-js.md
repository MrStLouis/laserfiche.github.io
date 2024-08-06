<!--© 2024 Laserfiche.
See LICENSE-DOCUMENTATION and LICENSE-CODE in the project root for license information.-->

```javascript
/**
 * Returns a list of repositories. Errors if no repositories are found.
 * @param {RepositoryApiClient} apiClient 
 * @returns {Promise<Repository[]>}
 * @preserve
 */
const getRepositories = async (apiClient) => {
  const resolvedApiClient = apiClient ?? await LFForm.getLaserficheAPIClient("Default");
  const repositories = await resolvedApiClient.repositoriesClient.listRepositories({});
  if (!repositories.value || repositories.value.length === 0) {
    throw new Error("No repositories found");
  }
  return repositories.value;
};

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
const patchEntryMetadata = async ({
  options,
  ...apiOptions
}) => {
  const { apiClient, repositoryId, entryId } = await resolveDefaultRepositoryAPIOptions(apiOptions);
  const metadataFields = await apiClient.entriesClient.listFields({
    repositoryId,
    entryId,
    formatFieldValues: false
  });
  const metadataMap = metadataFields.value?.reduce((acc, field) => {
    if (!field.name) return acc;
    acc[field.name] = { name: field.name, values: field.values };
    return acc;
  }, {});
  if (!metadataMap) return;
  for (const formField of options.newMetadata) {
    const metadataField = metadataMap[formField.settings.label];
    if (!metadataField) continue;
    const value = LFForm.getFieldValues(formField);
    if (!value) continue;
    if (typeof value === "string" || typeof value === "number") {
      metadataMap[formField.settings.label].values = [value.toString()];
    }
  }
  const request = {
    fields: Object.values(metadataMap)
  };
  const patchResponse = apiClient.entriesClient.setFields({
    repositoryId,
    entryId,
    request
  });
  return patchResponse;
};

const main = async () => {
  const formFields = {
    saveButton: 3,
    entryId: "Entry_ID"
  };
  await LFForm.changeFieldSettings({ fieldId: formFields.saveButton }, {
    content: `<button class="btn btn-primary float-right" onclick="window.onSaveClick()">Save</button>`
  });
  const apiClient = await LFForm.getLaserficheAPIClient("Default");
  const repositories = await getRepositories(apiClient);
  const repository = repositories[0];
  const entryIdField = LFForm.findFieldsByVariableName(formFields.entryId)[0];
  window.onSaveClick = async () => {
    await patchEntryMetadata({
      repositoryId: repository.id,
      entryIdField,
      apiClient,
      options: {
        newMetadata: LFForm.findFieldsByClassName("new-metadata")
      }
    });
  };
};
void main();
//# sourceMappingURL=Update.js.map

```