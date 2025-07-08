# zod-cheat-sheet


# Classes

## Properties

### Static Schema
```typescript
// ==== Imports ====
import { z } from 'zod'

export class IndexingPipelineService {
    /**
     * ZOD Schema for validating index names according to database naming conventions.
     * - Must start with a letter (a-z, A-Z) or underscore (_)
     * - Can contain only letters, numbers, and underscores
     * - Maximum length of 63 characters
     */
    private static readonly _indexNameSchema = z
        .string()
        .min(1, 'Index name cannot be empty')
        .max(63, 'Index name must be at most 63 characters long')
        .regex(
            /^[a-zA-Z_][a-zA-Z0-9_]*$/,
            'Index name must start with a letter or underscore and contain only letters, numbers, or underscores'
        )

    /**
     * Validates an index name using ZOD schema validation.
     * Throws a ZodError if validation fails with detailed error information.
     * 
     * @param indexName - The index name to validate
     * @throws {ZodError} When the index name doesn't meet validation requirements
     */
    private static _validateIndexName(indexName: Readonly<string>): void {
        IndexingPipelineService._indexNameSchema.parse(indexName)
    }
}
```
