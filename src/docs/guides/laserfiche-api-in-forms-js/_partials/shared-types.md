<!--© 2024 Laserfiche.
See LICENSE-DOCUMENTATION and LICENSE-CODE in the project root for license information.-->

- `entryIdField` (*optional*): The entry ID to map to the form. This can be a field, a string, or a number. If empty it will attempt to find the field by variable name. Looks for `EntryId` or `Entry_ID`.
- `repositoryId` (*optional*): A string representing repository ID to map to the form. If not provided, the function will attempt to get the default repository.
- `apiClient` (*optional*): The authenticated API client. If not provided, the function will attempt to get the default client.