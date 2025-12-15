# @manifesto-ai/projection-ui

[![npm](https://img.shields.io/npm/v/@manifesto-ai/projection-ui)](https://www.npmjs.com/package/@manifesto-ai/projection-ui)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> Framework-agnostic UI state projection for Manifesto AI

Transform domain policies into UI states automatically. The domain decides **what** (relevant, editable, required), and your UI decides **how** (visible, enabled, styled).

## Installation

```bash
pnpm add @manifesto-ai/projection-ui @manifesto-ai/core
```

## Core Concept

Domain policies are projected to UI states through a simple mapping:

| Domain Policy | UI State | Description |
|--------------|----------|-------------|
| `relevance` | `visible` | Should this field be shown? |
| `editability` | `enabled` | Can the user interact with it? |
| `requirement` | `required` | Must the user provide a value? |

```typescript
import { projectFieldPolicy } from '@manifesto-ai/projection-ui';

const uiState = projectFieldPolicy(policy, runtime);
// {
//   visible: true,
//   enabled: false,
//   required: true,
//   validation: { valid: true, issues: [] },
//   meta: { path: 'data.email', label: 'Email' }
// }
```

## Quick Start

### Single Field Projection

```typescript
import { projectFieldPolicy } from '@manifesto-ai/projection-ui';
import { fieldPolicy, condition } from '@manifesto-ai/core';

const emailPolicy = fieldPolicy({
  relevance: true,
  editability: condition({ $not: { $get: 'state.isSubmitting' } }),
  requirement: true,
});

const state = projectFieldPolicy(emailPolicy, runtime, 'data.email');

if (state.visible) {
  renderField({
    disabled: !state.enabled,
    required: state.required,
    error: state.validation.issues[0]?.message,
  });
}
```

### Multiple Fields with ProjectionManager

```typescript
import { createProjectionManager } from '@manifesto-ai/projection-ui';

const manager = createProjectionManager({
  runtime,
  domain,
  fields: {
    paths: ['data.name', 'data.email', 'data.phone'],
  },
  actions: {
    actionIds: ['submit', 'save', 'cancel'],
  },
});

// Get current state
const emailState = manager.getFieldState('data.email');
const submitState = manager.getActionState('submit');

// Subscribe to changes
const unsubscribe = manager.subscribeFields((states, changedPaths) => {
  changedPaths.forEach(path => {
    updateFieldUI(path, states.get(path));
  });
});

// Cleanup
manager.dispose();
```

## API Reference

### Field Projection

#### `projectFieldPolicy(policy, runtime, path?)`

Projects a single field policy to UI state.

```typescript
const state = projectFieldPolicy(policy, runtime, 'data.email');
```

#### `projectFieldPolicies(policies, runtime)`

Projects multiple field policies at once.

```typescript
const states = projectFieldPolicies({
  'data.name': namePolicy,
  'data.email': emailPolicy,
}, runtime);
// Map<string, UIFieldState>
```

#### Filter Utilities

```typescript
import {
  filterVisibleFields,
  filterEnabledFields,
  getRequiredFields,
} from '@manifesto-ai/projection-ui';

const visible = filterVisibleFields(states);   // Only visible fields
const enabled = filterEnabledFields(states);   // Only enabled fields
const required = getRequiredFields(states);    // Only required fields
```

### Action Projection

#### `projectActionState(actionId, runtime)`

Projects an action's availability state.

```typescript
const state = projectActionState('submit', runtime);
// {
//   actionId: 'submit',
//   available: true,
//   unavailableReason: null,
//   executing: false,
//   preconditions: [
//     { conditionId: 'hasItems', satisfied: true },
//     { conditionId: 'isValid', satisfied: true },
//   ]
// }
```

#### Action Utilities

```typescript
import {
  getAvailableActions,
  getUnavailableActions,
  setExecuting,
} from '@manifesto-ai/projection-ui';

const available = getAvailableActions(states);     // ['save', 'cancel']
const unavailable = getUnavailableActions(states); // [{ actionId: 'submit', reason: '...' }]
const executing = setExecuting(state, true);       // Mark as executing
```

### Event Projection

Create UI events (toasts, notifications) from domain events:

```typescript
import {
  createSuccessToast,
  createErrorToast,
  projectEvent,
} from '@manifesto-ai/projection-ui';

// Quick toast creation
const toast = createSuccessToast('Saved', 'Your changes have been saved');

// Or project from domain events
const uiEvent = projectEvent(domainEvent, {
  transformer: (event) => ({
    title: 'Order Created',
    message: `Order #${event.payload.id} confirmed`,
    severity: 'success',
  }),
});

manager.emitEvent(toast);
```

### ProjectionManager

Full-featured manager for coordinating field, action, and event projections:

```typescript
interface ProjectionManager {
  // Fields
  getFieldState(path: string): UIFieldState | undefined;
  getAllFieldStates(): Map<string, UIFieldState>;
  subscribeFields(listener: FieldStateListener): () => void;

  // Actions
  getActionState(actionId: string): UIActionState | undefined;
  getAllActionStates(): Map<string, UIActionState>;
  subscribeActions(listener: ActionStateListener): () => void;
  setActionExecuting(actionId: string, executing: boolean): void;

  // Events
  emitEvent(event: UIEvent): void;
  getPendingEvents(): UIEvent[];
  dismissEvent(eventId: string): void;
  subscribeEvents(listener: UIEventListener): () => void;

  // Lifecycle
  dispose(): void;
}
```

## Framework Integration

This package provides **framework-agnostic** projection logic. For framework-specific bindings, create thin wrappers in your application or use bridge packages:

### React Example

```typescript
function useFieldState(manager: ProjectionManager, path: string) {
  const [state, setState] = useState(() => manager.getFieldState(path));

  useEffect(() => {
    return manager.subscribeFields((states, changed) => {
      if (changed.includes(path)) {
        setState(states.get(path));
      }
    });
  }, [manager, path]);

  return state;
}
```

### Vue Example

```typescript
function useFieldState(manager: ProjectionManager, path: string) {
  const state = ref(manager.getFieldState(path));

  onMounted(() => {
    const unsubscribe = manager.subscribeFields((states, changed) => {
      if (changed.includes(path)) {
        state.value = states.get(path);
      }
    });
    onUnmounted(unsubscribe);
  });

  return state;
}
```

## Types

```typescript
interface UIFieldState {
  visible: boolean;
  enabled: boolean;
  required: boolean;
  validation: {
    valid: boolean;
    issues: ValidationIssue[];
  };
  meta: {
    path: string;
    label?: string;
  };
}

interface UIActionState {
  actionId: string;
  available: boolean;
  unavailableReason: string | null;
  executing: boolean;
  preconditions: UIPreconditionState[];
}

interface UIEvent {
  id: string;
  title: string;
  message?: string;
  severity: 'success' | 'error' | 'warning' | 'info';
  channel?: string;
  timestamp: number;
  dismissed: boolean;
}
```

## License

MIT © [Manifesto AI](https://github.com/manifesto-ai)
