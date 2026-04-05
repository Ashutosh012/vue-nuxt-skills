# 🟢 Vue 3 + Nuxt 3 Cursor Skills

On-demand AI knowledge packages for Cursor — deep expertise for Vue 3 and Nuxt 3, loaded only when relevant to your task.

> **Skills vs Rules:** Rules are always-on constraints. Skills are deep knowledge packages loaded on-demand when your task matches the skill's description. This keeps Cursor's context lean and focused.

---

## 📦 Skills Included

| Skill | Triggers When You Ask About... |
|---|---|
| **Vue 3** | Components, composables, Pinia stores, Vue Router, reactivity, performance |
| **Nuxt 3** | Pages, server routes, data fetching, middleware, SEO, plugins, rendering strategy |

---

## 🚀 Installation

Copy the `.cursor/` folder into your project root:

```bash
# Clone this repo
https://github.com/Ashutosh012/vue-nuxt-skills.git

# Copy into your Vue or Nuxt project
cp -r vue-nuxt-cursor-skills/.cursor /path/to/your/project/
```

That's it. Cursor will auto-discover the skills.

---

## 📁 File Structure

```
.cursor/
└── skills/
    ├── vue3/
    │   └── SKILL.md      # Vue 3 Composition API deep knowledge
    └── nuxt3/
        └── SKILL.md      # Nuxt 3 full-stack deep knowledge
```

---

## 🧠 What Each Skill Covers

### Vue 3 Skill
- Complete `<script setup lang="ts">` component skeleton
- Composable template with loading/error/data states
- Pinia Setup Store pattern with `readonly()` state
- Vue Router typed routes + navigation guards
- Reactivity cheat sheet (`ref`, `reactive`, `computed`, `watch`, `shallowRef`, `markRaw`)
- Common patterns: async components, `v-model`, `provide/inject`, `Teleport`
- Performance checklist

### Nuxt 3 Skill
- Full project structure reference
- Production-ready `nuxt.config.ts` template
- Data fetching decision tree (`useFetch` vs `useAsyncData` vs `$fetch`)
- Server route templates with Zod validation
- Page + layout templates
- Named and global middleware patterns
- Plugin templates (client-only and universal)
- SEO with `useSeoMeta` + JSON-LD structured data
- Rendering strategy reference (SSR / SSG / ISR / CSR)
- Anti-patterns table

---

## 📤 Publishing to cursor.directory

cursor.directory accepts single-file `.cursorrules` format. To publish these skills there:

1. **Combine skills into `.cursorrules`** — paste the SKILL.md content of both skills into one `.cursorrules` file at the project root (a combined version is at the bottom of this README).
2. **Go to [cursor.directory](https://cursor.directory)** and click Submit.
3. **Paste your `.cursorrules`** content into the form.
4. **Add tags:** `vue`, `nuxt`, `vue3`, `nuxt3`, `typescript`, `composition-api`, `pinia`, `nitro`.
5. Submit and it will be listed publicly for the community.

Also consider submitting to **[awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules)**:
- Fork the repo
- Create folder: `vue3-nuxt3-cursorrules-prompt-file/`
- Add your `.cursorrules` file inside
- Open a Pull Request

---

## 🤝 Contributing

PRs welcome. To add a new skill (e.g. VueUse, Vitest, Tailwind + Vue):
1. Create `.cursor/skills/your-skill/SKILL.md`
2. Follow the existing SKILL.md format (Description → Workflow → Templates → Cheat Sheet)
3. Open a PR

---

## 📄 License

MIT
