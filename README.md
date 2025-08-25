# zod-cheat-sheet



# Schemas

## Properties

### Readonly



<details><summary>Click to expand..</summary>

# Zod Date-Felder readonly-konform transformieren für Parameter-Type-Safety

## ❗ Critical Rules

- **MUST** verwende `transform<Readonly<Date>>(d => d)` für alle Date-Felder in Zod-Schemas
- **MUST** verwende `schema.readonly()` + `z.output<typeof schema>` für Parameter-Typen
- **MUST** verwende `.nullable()` und `.optional()` VOR dem `.transform()` - niemals danach
- **NEVER** verwende Type-Casts oder ESLint-Disable-Kommentare als Workaround
- **NEVER** wechsle zu ISO-Strings nur wegen readonly-Compliance
- **ALWAYS** bewahre Date als echte Date-Objekte

## 📋 Problem-Übersicht

**Ausgangssituation:** ESLint-Regel `@typescript-eslint/prefer-readonly-parameter-types` schlägt fehl bei Funktionsparametern, die Zod-Schema-Typen mit Date-Feldern enthalten, weil Date inherent mutierbar ist.

### **🚫 Das Problem:**
- **Date ist mutierbar** → ESLint meckert bei Parameter-Types
- **Zod `.readonly()` macht nur Keys readonly** → Date bleibt mutierbar
- **Typische Lösungsversuche scheitern** → Type-Casts, ESLint-Disable, ISO-String-Konvertierung

## ✅ Examples

<example>
// ✅ KORREKT - Zod Date readonly-konform mit transform<Readonly<Date>>

import { z } from 'zod'

// 1) Date-Felder explizit als immutable typisieren
const pvsPatientDataSchema = z.object({
  createdAt: z.date().transform<Readonly<Date>>((date: Readonly<Date>) => date).optional(),
  changedAt: z.date().transform<Readonly<Date>>((date: Readonly<Date>) => date).nullable().optional(),
}).strict()

// 2) Readonly-Instanz ableiten und Output-Typ daraus gewinnen
const pvsPatientDataSchemaRO = pvsPatientDataSchema.readonly()
type PvsPatient = z.output<typeof pvsPatientDataSchemaRO>

// 3) Verwendung: Parameter ist jetzt readonly-konform, Date bleibt Date
export function handlePatient(input: PvsPatient): void {
  // ✅ ESLint @typescript-eslint/prefer-readonly-parameter-types: OK
  // ✅ Date-Felder sind Readonly<Date> - keine mutierenden Methoden
  // ✅ Date bleibt fachlich Date - keine String-Konvertierung
  console.log(input.createdAt?.getFullYear()) // ✅ Getter funktionieren
  // input.createdAt?.setFullYear(2024) // ❌ Mutating methods nicht verfügbar
}

// ✅ WARUM DAS FUNKTIONIERT:
// - transform<Readonly<Date>>(d => d) entfernt muting set* APIs aus Type
// - schema.readonly() + z.output<typeof schema> liefert readonly Parameter-Typ
// - Date bleibt zur Runtime echtes Date-Objekt
// - Keine Type-Casts, keine ESLint-Disable, keine String-Konvertierung
</example>

<example type="invalid">
// ❌ PROBLEM - Standard Zod-Schema ohne readonly-Compliance

import { z } from 'zod'

const pvsPatientDataSchema = z.object({
  createdAt: z.date().optional(),
  changedAt: z.date().nullable().optional(),
}).strict()

type PvsPatient = z.infer<typeof pvsPatientDataSchema>

// ❌ ESLint-Fehler: Parameter ist nicht tief-immutable
export function handlePatient(input: PvsPatient): void {
  // ESLint Error: @typescript-eslint/prefer-readonly-parameter-types
  // "Parameter 'input' should be a read-only type. 
  //  Its type 'PvsPatient' is mutable."
}

// ❌ FEHLGESCHLAGENE LÖSUNGSVERSUCHE:

// Versuch 1: .readonly() ohne transform - Date bleibt mutierbar
const badSchema1 = pvsPatientDataSchema.readonly()
type BadType1 = z.output<typeof badSchema1> // Date ist immer noch mutierbar

// Versuch 2: Type-Cast Workaround - Anti-Pattern
export function handlePatientBad(input: PvsPatient as Readonly<PvsPatient>): void {
  // ❌ Type-Cast umgeht Typsicherheit, löst Problem nicht
}

// Versuch 3: ESLint-Disable - Anti-Pattern  
// eslint-disable-next-line @typescript-eslint/prefer-readonly-parameter-types
export function handlePatientDisabled(input: PvsPatient): void {
  // ❌ ESLint-Regel deaktiviert, Problem nicht gelöst
}

// Versuch 4: ISO-String statt Date - Verliert Date-Semantik
const stringSchema = z.object({
  createdAt: z.string().datetime().optional(),
  changedAt: z.string().datetime().nullable().optional(),
})
// ❌ Date-Funktionalität geht verloren, nur Strings verfügbar
</example>

## 🎯 **Warum diese Lösung optimal ist**

| Kriterium | Bewertung | Begründung |
|-----------|-----------|------------|
| **🛡️ Type Safety** | ✅ **Perfekt** | Readonly<Date> eliminiert muting APIs, behält Getter |
| **⚡ Performance** | ✅ **Optimal** | Keine Runtime-Kosten, nur Compile-time Type-Transformation |
| **🔧 Wartbarkeit** | ✅ **Hoch** | Minimale Schema-Änderung, keine externen Workarounds |
| **📖 Lesbarkeit** | ✅ **Klar** | Explizite Intention durch transform<Readonly<Date>> |
| **🎯 Semantik** | ✅ **Erhalten** | Date bleibt Date, keine String-Konvertierung |
| **🔗 ESLint-Compliance** | ✅ **Vollständig** | Keine Disable-Kommentare oder Type-Casts |

Die `transform<Readonly<Date>>()` Transformation ist **minimal invasiv** und beeinflusst andere Schema-Definitionen nicht.

</details>



















<br><br>

--- 

<br><br>




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
