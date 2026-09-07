# Unit test — reference

Companion to [SKILL.md](SKILL.md). Concrete Go before/after patterns.

---

## Before / after (nested `t.Run` → table)

### Before (harder to scan)

```go
func TestResolveAppRegisterSource(t *testing.T) {
	ctx := context.Background()

	t.Run("known organic key", func(t *testing.T) {
		assert.Equal(t, organic, ResolveAppRegisterSource(ctx, true, "instagram"))
	})
	t.Run("known paid key", func(t *testing.T) {
		assert.Equal(t, paid, ResolveAppRegisterSource(ctx, true, "googleadwords_int"))
	})
	t.Run("unknown defaults paid", func(t *testing.T) {
		assert.Equal(t, paid, ResolveAppRegisterSource(ctx, true, "newpartner_int"))
	})
}
```

### After (checklist)

```go
func TestResolveAppRegisterSource(t *testing.T) {
	ctx := context.Background()

	tests := []struct {
		name        string
		hasTracker  bool
		mediaSource string
		want        string
	}{
		{name: "known organic key", hasTracker: true, mediaSource: "instagram", want: organic},
		{name: "known paid key", hasTracker: true, mediaSource: "googleadwords_int", want: paid},
		{name: "unknown defaults paid", hasTracker: true, mediaSource: "newpartner_int", want: paid},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got := ResolveAppRegisterSource(ctx, tt.hasTracker, tt.mediaSource)
			assert.Equal(t, tt.want, got)
		})
	}
}
```

---

## Valid + invalid in one table

```go
func TestParseMediaSourceSimplifiedJSON(t *testing.T) {
	tests := []struct {
		name    string
		raw     []byte
		want    map[string]string
		wantErr string
	}{
		{
			name: "valid json",
			raw:  []byte(`{"instagram":"App - Register Organic"}`),
			want: map[string]string{"instagram": "App - Register Organic"},
		},
		{
			name:    "invalid json",
			raw:     []byte(`{invalid`),
			wantErr: "parse media_source_simplified.json",
		},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got, err := parseMediaSourceSimplifiedJSON(tt.raw)
			if tt.wantErr != "" {
				require.Error(t, err)
				assert.Contains(t, err.Error(), tt.wantErr)
				return
			}
			require.NoError(t, err)
			assert.Equal(t, tt.want, got)
		})
	}
}
```

---

## Map / fixture smoke as a small table

When loading a real embedded JSON, a key→value table beats four separate asserts:

```go
func TestLoadMediaSourceMap_EmbeddedJSON(t *testing.T) {
	tests := []struct {
		name string
		key  string
		want string
	}{
		{name: "instagram organic", key: "instagram", want: "App - Register Organic"},
		{name: "googleadwords paid", key: "googleadwords_int", want: "App - Register Paid"},
	}

	sourceMap, err := loadMediaSourceMap()
	require.NoError(t, err)
	require.NotEmpty(t, sourceMap)

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			assert.Equal(t, tt.want, sourceMap[tt.key])
		})
	}
}
```

---

## Anti-patterns

| Avoid | Prefer |
|-------|--------|
| One `TestFoo_Bar` per tiny case with copy-pasted setup | One `TestFoo` + table rows |
| Multiple unrelated inputs asserted inside one `t.Run` | One row per input |
| Silent skip of old scenarios during a “cleanup” rewrite | Preserve every prior case |
| Changing production defaults to make tests prettier | Fix tests to match agreed behavior |
| Running the entire monorepo test suite for a one-file change | Focused `-run` / path filter |

---

## Other languages (same idea)

| Tool | Table-driven shape |
|------|--------------------|
| Go | `tests := []struct{...}` + `t.Run` |
| Jest / Vitest | `it.each([...])('...', (a, b) => { ... })` |
| pytest | `@pytest.mark.parametrize("input,want", [...])` |

Always prefer the form already used in that package.
