# Imperium Moda Social Masculina — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a premium landing page + catalog page for Imperium Moda Social Masculina, plus add "roupas" type to the existing cardapio-admin panel.

**Architecture:** Static HTML/CSS/JS frontend (Marieta pattern) fetching product data from Firestore REST API. Admin panel (React + Firebase) extended with new "roupas" type and RoupasEditor page. Seed script populates 33 initial products.

**Tech Stack:** HTML/CSS/JS vanilla (frontend), React + Firebase + Tailwind (admin), Firestore REST API (data), Google Fonts (Playfair Display + Inter)

**Spec:** `docs/superpowers/specs/2026-04-06-imperium-moda-social-design.md`

---

## File Structure

### Frontend (`C:\dev\prototipos\imperium moda social masculina\`)

| File | Purpose |
|---|---|
| `index.html` | Landing page — 10 sections + header/footer, all CSS inline, JS inline + external fallback |
| `catalogo.html` | Catalog page — banner, filters, product grid, modal, shared CSS/JS with index |
| `roupas.js` | Fallback product data (33 products hardcoded for offline/file:// use) |

### Admin (`C:\dev\cardapio-admin\`)

| File | Action | Purpose |
|---|---|---|
| `src/App.jsx` | Modify | Add route for RoupasEditor |
| `src/pages/AdminRestaurantes.jsx` | Modify | Add "roupas" type option + initialization |
| `src/pages/Dashboard.jsx` | Modify | Add "roupas" type buttons |
| `src/pages/RoupasEditor.jsx` | Create | Product editor (follows VeiculosEditor pattern) |

### Seed Script (`C:\dev\cardapio-admin\`)

| File | Purpose |
|---|---|
| `seed-imperium.js` | Inserts 33 products + businessInfo into Firestore |

---

## Task 1: cardapio-admin — Add "roupas" type to AdminRestaurantes

**Files:**
- Modify: `C:\dev\cardapio-admin\src\pages\AdminRestaurantes.jsx`

- [ ] **Step 1: Add "roupas" radio option to the type selector**

In `AdminRestaurantes.jsx`, after the "garagem" radio label (line 197), add the "roupas" option:

```jsx
              <label className={`flex-1 flex items-center gap-2 p-3 rounded-xl border-2 cursor-pointer transition ${form.type === 'roupas' ? 'border-rose-500 bg-rose-50' : 'border-slate-200 hover:border-slate-300'}`}>
                <input
                  type="radio"
                  name="type"
                  value="roupas"
                  checked={form.type === 'roupas'}
                  onChange={e => setForm(prev => ({ ...prev, type: e.target.value }))}
                  className="accent-rose-500"
                />
                <div>
                  <span className="text-sm font-medium text-slate-800">👔 Loja de Roupas</span>
                  <p className="text-[11px] text-slate-400">Catálogo e informações</p>
                </div>
              </label>
```

- [ ] **Step 2: Add "roupas" initialization in handleCreate**

In `handleCreate`, after the `if (form.type === 'garagem')` block (line 75) and before the `else` block (line 76), add:

```jsx
      } else if (form.type === 'roupas') {
        await setDoc(doc(db, 'restaurants', form.slug, 'data', 'roupas'), {
          content: [],
          updatedAt: new Date().toISOString()
        })
        await setDoc(doc(db, 'restaurants', form.slug, 'data', 'businessInfo'), {
          content: {
            name: form.name,
            city: '',
            slogan: '',
            tagline: 'Moda Masculina Premium',
            whatsapp: '',
            whatsappNumber: '',
            phone: '',
            address: '',
            neighborhood: '',
            cityState: '',
            cep: '',
            hours: { weekdays: '', saturday: '' },
            instagram: '',
            facebook: '',
            googleMapsEmbed: '',
            googleMapsLink: ''
          },
          updatedAt: new Date().toISOString()
        })
```

- [ ] **Step 3: Add "roupas" deletion logic in handleDelete**

In `handleDelete`, after `if (type === 'garagem')` block (line 105) and before the `else` block, add:

```jsx
      } else if (type === 'roupas') {
        await deleteDoc(doc(db, 'restaurants', slug, 'data', 'roupas'))
        await deleteDoc(doc(db, 'restaurants', slug, 'data', 'businessInfo'))
```

- [ ] **Step 4: Update type helpers**

Replace the helpers at lines 117-121:

```jsx
  const isGaragem = (r) => r.type === 'garagem'
  const isRoupas = (r) => r.type === 'roupas'
  const typeLabel = (r) => isGaragem(r) ? 'Garagem' : isRoupas(r) ? 'Roupas' : 'Restaurante'
  const typeBadgeClass = (r) => isGaragem(r)
    ? 'bg-blue-50 text-blue-700'
    : isRoupas(r)
      ? 'bg-rose-50 text-rose-700'
      : 'bg-amber-50 text-amber-700'
```

- [ ] **Step 5: Add "roupas" action buttons in the restaurant list**

In the restaurant list JSX (around lines 252-283), update the conditional rendering to handle the "roupas" type. Replace the entire `{isGaragem(r) ? ( ... ) : ( ... )}` block with:

```jsx
              {isGaragem(r) ? (
                <>
                  <Link
                    to={`/restaurante/${r.slug}/veiculos`}
                    className="text-xs px-3 py-1.5 bg-blue-50 text-blue-700 rounded-lg hover:bg-blue-100 transition"
                  >
                    Veículos
                  </Link>
                  <Link
                    to={`/restaurante/${r.slug}/info`}
                    className="text-xs px-3 py-1.5 bg-teal-50 text-teal-700 rounded-lg hover:bg-teal-100 transition"
                  >
                    Informações
                  </Link>
                </>
              ) : isRoupas(r) ? (
                <>
                  <Link
                    to={`/restaurante/${r.slug}/roupas`}
                    className="text-xs px-3 py-1.5 bg-rose-50 text-rose-700 rounded-lg hover:bg-rose-100 transition"
                  >
                    Catálogo
                  </Link>
                  <Link
                    to={`/restaurante/${r.slug}/info`}
                    className="text-xs px-3 py-1.5 bg-teal-50 text-teal-700 rounded-lg hover:bg-teal-100 transition"
                  >
                    Informações
                  </Link>
                </>
              ) : (
                <>
                  <Link
                    to={`/restaurante/${r.slug}/cardapio`}
                    className="text-xs px-3 py-1.5 bg-amber-50 text-amber-700 rounded-lg hover:bg-amber-100 transition"
                  >
                    Cardápio
                  </Link>
                  <Link
                    to={`/restaurante/${r.slug}/promocoes`}
                    className="text-xs px-3 py-1.5 bg-purple-50 text-purple-700 rounded-lg hover:bg-purple-100 transition"
                  >
                    Promoções
                  </Link>
                </>
              )}
```

- [ ] **Step 6: Verify admin builds**

```bash
cd C:\dev\cardapio-admin && npm run build
```

Expected: Build succeeds with no errors.

- [ ] **Step 7: Commit**

```bash
cd C:\dev\cardapio-admin
git add src/pages/AdminRestaurantes.jsx
git commit -m "feat: add 'roupas' client type to AdminRestaurantes"
```

---

## Task 2: cardapio-admin — Update Dashboard for "roupas" type

**Files:**
- Modify: `C:\dev\cardapio-admin\src\pages\Dashboard.jsx`

- [ ] **Step 1: Add isRoupas helper**

After line 33 (`const isGaragem = (r) => r.type === 'garagem'`), add:

```jsx
  const isRoupas = (r) => r.type === 'roupas'
```

- [ ] **Step 2: Update type badge rendering**

Replace line 70-72 (the span with the badge):

```jsx
                <span className={`text-[10px] font-semibold px-2 py-0.5 rounded-full ${isGaragem(r) ? 'bg-blue-50 text-blue-700' : isRoupas(r) ? 'bg-rose-50 text-rose-700' : 'bg-amber-50 text-amber-700'}`}>
                  {isGaragem(r) ? '🚗 Garagem' : isRoupas(r) ? '👔 Roupas' : '🍽️ Restaurante'}
                </span>
```

- [ ] **Step 3: Add "roupas" buttons in the conditional rendering**

Replace the entire `{isGaragem(r) ? ( ... ) : ( ... )}` block (lines 76-106) with:

```jsx
                {isGaragem(r) ? (
                  <>
                    <Link
                      to={`/restaurante/${r.slug}/veiculos`}
                      className="flex-1 text-center py-2.5 bg-blue-50 hover:bg-blue-100 text-blue-700 rounded-xl text-sm font-medium transition"
                    >
                      Veículos
                    </Link>
                    <Link
                      to={`/restaurante/${r.slug}/info`}
                      className="flex-1 text-center py-2.5 bg-teal-50 hover:bg-teal-100 text-teal-700 rounded-xl text-sm font-medium transition"
                    >
                      Informações
                    </Link>
                  </>
                ) : isRoupas(r) ? (
                  <>
                    <Link
                      to={`/restaurante/${r.slug}/roupas`}
                      className="flex-1 text-center py-2.5 bg-rose-50 hover:bg-rose-100 text-rose-700 rounded-xl text-sm font-medium transition"
                    >
                      Catálogo
                    </Link>
                    <Link
                      to={`/restaurante/${r.slug}/info`}
                      className="flex-1 text-center py-2.5 bg-teal-50 hover:bg-teal-100 text-teal-700 rounded-xl text-sm font-medium transition"
                    >
                      Informações
                    </Link>
                  </>
                ) : (
                  <>
                    <Link
                      to={`/restaurante/${r.slug}/cardapio`}
                      className="flex-1 text-center py-2.5 bg-amber-50 hover:bg-amber-100 text-amber-700 rounded-xl text-sm font-medium transition"
                    >
                      Cardapio
                    </Link>
                    <Link
                      to={`/restaurante/${r.slug}/promocoes`}
                      className="flex-1 text-center py-2.5 bg-purple-50 hover:bg-purple-100 text-purple-700 rounded-xl text-sm font-medium transition"
                    >
                      Promocoes
                    </Link>
                  </>
                )}
```

- [ ] **Step 4: Verify build**

```bash
cd C:\dev\cardapio-admin && npm run build
```

- [ ] **Step 5: Commit**

```bash
cd C:\dev\cardapio-admin
git add src/pages/Dashboard.jsx
git commit -m "feat: add 'roupas' type buttons to Dashboard"
```

---

## Task 3: cardapio-admin — Create RoupasEditor page

**Files:**
- Create: `C:\dev\cardapio-admin\src\pages\RoupasEditor.jsx`

- [ ] **Step 1: Create RoupasEditor.jsx**

Create `C:\dev\cardapio-admin\src\pages\RoupasEditor.jsx` following the VeiculosEditor pattern exactly, adapted for clothing products. The file reuses the `ImageUploader` pattern from VeiculosEditor.

```jsx
import { useState, useEffect, useRef } from 'react'
import { useParams, Link } from 'react-router-dom'
import { db, storage } from '../firebase'
import { doc, getDoc, setDoc } from 'firebase/firestore'
import { ref, uploadBytesResumable, getDownloadURL, deleteObject } from 'firebase/storage'

function generateId() {
  return Math.random().toString(36).substring(2, 9)
}

const CATEGORIAS = [
  { value: 'camisas-sociais', label: 'Camisas Sociais' },
  { value: 'camisetas-polos', label: 'Camisetas & Polos' },
  { value: 'calcas-bermudas', label: 'Calças & Bermudas' },
  { value: 'ternos-costumes', label: 'Ternos & Costumes' },
  { value: 'acessorios', label: 'Acessórios' },
  { value: 'kits-combinacoes', label: 'Kits & Combinações' },
  { value: 'jaquetas-sueteres', label: 'Jaquetas & Suéteres' },
  { value: 'perfumes', label: 'Perfumes' },
]

// ===== Image Uploader (até 6 fotos por produto) =====
function ImageUploader({ images = [], onChange, slug, productId }) {
  const [uploading, setUploading] = useState(false)
  const [progress, setProgress] = useState(0)
  const fileRef = useRef()

  async function handleUpload(e) {
    const file = e.target.files?.[0]
    if (!file) return
    if (images.length >= 6) return alert('Máximo de 6 fotos por produto.')

    setUploading(true)
    setProgress(0)

    const storagePath = `restaurants/${slug}/roupas/${productId}/${Date.now()}_${file.name}`
    const storageRef = ref(storage, storagePath)
    const task = uploadBytesResumable(storageRef, file)

    task.on('state_changed',
      (snap) => setProgress(Math.round((snap.bytesTransferred / snap.totalBytes) * 100)),
      (err) => { alert('Erro no upload: ' + err.message); setUploading(false) },
      async () => {
        const url = await getDownloadURL(task.snapshot.ref)
        onChange([...images, { url, storagePath }])
        setUploading(false)
        setProgress(0)
        if (fileRef.current) fileRef.current.value = ''
      }
    )
  }

  async function handleRemove(index) {
    const img = images[index]
    try {
      if (img.storagePath) {
        await deleteObject(ref(storage, img.storagePath))
      }
    } catch (err) {
      console.warn('Erro ao deletar do Storage:', err)
    }
    onChange(images.filter((_, i) => i !== index))
  }

  function handleReorder(fromIndex, toIndex) {
    const updated = [...images]
    const [moved] = updated.splice(fromIndex, 1)
    updated.splice(toIndex, 0, moved)
    onChange(updated)
  }

  return (
    <div>
      <label className="block text-[11px] font-medium text-slate-500 mb-2">
        Fotos ({images.length}/6)
      </label>
      <div className="flex flex-wrap gap-2 mb-2">
        {images.map((img, i) => (
          <div key={i} className="relative group w-24 h-18 rounded-lg overflow-hidden border border-slate-200 bg-slate-100">
            <img
              src={typeof img === 'string' ? img : img.url}
              alt={`Foto ${i + 1}`}
              className="w-full h-full object-cover"
            />
            <span className="absolute top-1 left-1 bg-black/60 text-white text-[9px] font-bold px-1.5 py-0.5 rounded">
              {i + 1}
            </span>
            <div className="absolute inset-0 bg-black/0 group-hover:bg-black/40 transition flex items-center justify-center gap-1 opacity-0 group-hover:opacity-100">
              {i > 0 && (
                <button
                  type="button"
                  onClick={() => handleReorder(i, i - 1)}
                  className="w-6 h-6 bg-white/90 rounded-full flex items-center justify-center text-slate-700 text-xs hover:bg-white"
                >←</button>
              )}
              <button
                type="button"
                onClick={() => handleRemove(i)}
                className="w-6 h-6 bg-red-500 rounded-full flex items-center justify-center text-white text-xs hover:bg-red-600"
              >✕</button>
              {i < images.length - 1 && (
                <button
                  type="button"
                  onClick={() => handleReorder(i, i + 1)}
                  className="w-6 h-6 bg-white/90 rounded-full flex items-center justify-center text-slate-700 text-xs hover:bg-white"
                >→</button>
              )}
            </div>
          </div>
        ))}
        {images.length < 6 && !uploading && (
          <label className="w-24 h-18 rounded-lg border-2 border-dashed border-slate-300 hover:border-rose-400 flex flex-col items-center justify-center cursor-pointer transition bg-slate-50 hover:bg-rose-50">
            <svg className="w-5 h-5 text-slate-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
              <path strokeLinecap="round" strokeLinejoin="round" d="M12 4v16m8-8H4" />
            </svg>
            <span className="text-[10px] text-slate-400 mt-0.5">Foto</span>
            <input ref={fileRef} type="file" accept="image/*" onChange={handleUpload} className="hidden" />
          </label>
        )}
      </div>
      {uploading && (
        <div className="w-full bg-slate-100 rounded-full h-1.5 mb-1">
          <div className="bg-rose-500 h-1.5 rounded-full transition-all" style={{ width: `${progress}%` }} />
        </div>
      )}
    </div>
  )
}

// ===== Product Card (individual editor) =====
function ProductCard({ produto, onChange, onRemove, slug }) {
  const [collapsed, setCollapsed] = useState(false)
  const isSobConsulta = produto.preco === null || produto.preco === undefined

  function update(field, value) {
    onChange({ ...produto, [field]: value })
  }

  function toggleSobConsulta() {
    if (isSobConsulta) {
      update('preco', 0)
    } else {
      update('preco', null)
    }
  }

  const catLabel = CATEGORIAS.find(c => c.value === produto.categoria)?.label || produto.categoria || 'Sem categoria'

  return (
    <div className="bg-white rounded-2xl border border-slate-200 overflow-hidden">
      <div
        className="flex items-center justify-between p-4 cursor-pointer hover:bg-slate-50 transition"
        onClick={() => setCollapsed(!collapsed)}
      >
        <div className="flex items-center gap-3">
          {produto.images?.length > 0 && (
            <div className="w-12 h-12 rounded-lg overflow-hidden bg-slate-100 shrink-0">
              <img
                src={typeof produto.images[0] === 'string' ? produto.images[0] : produto.images[0]?.url}
                alt=""
                className="w-full h-full object-cover"
              />
            </div>
          )}
          <div>
            <h3 className="font-semibold text-slate-800 text-sm">
              {produto.nome || 'Novo produto'}
            </h3>
            <p className="text-xs text-slate-400">
              {catLabel}{produto.preco != null ? ` · R$ ${Number(produto.preco).toFixed(2).replace('.', ',')}` : ' · Sob consulta'}
            </p>
          </div>
        </div>
        <svg className={`w-4 h-4 text-slate-400 transition ${collapsed ? '' : 'rotate-180'}`} fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
          <path strokeLinecap="round" strokeLinejoin="round" d="M19 9l-7 7-7-7" />
        </svg>
      </div>

      {!collapsed && (
        <div className="p-4 pt-0 space-y-4 border-t border-slate-100">
          <ImageUploader
            images={produto.images || []}
            onChange={(imgs) => update('images', imgs)}
            slug={slug}
            productId={produto.id}
          />

          <div>
            <label className="block text-[11px] font-medium text-slate-500 mb-1">Nome do produto</label>
            <input
              type="text"
              value={produto.nome || ''}
              onChange={e => update('nome', e.target.value)}
              placeholder="Ex: Camisa Social Slim Fit"
              className="w-full px-3 py-2 rounded-lg border border-slate-200 text-sm focus:border-rose-500 outline-none"
            />
          </div>

          <div>
            <label className="block text-[11px] font-medium text-slate-500 mb-1">Descrição</label>
            <textarea
              value={produto.descricao || ''}
              onChange={e => update('descricao', e.target.value)}
              placeholder="Descreva o produto..."
              rows={3}
              className="w-full px-3 py-2 rounded-lg border border-slate-200 text-sm focus:border-rose-500 outline-none resize-none"
            />
          </div>

          <div className="grid grid-cols-2 gap-3">
            <div>
              <label className="block text-[11px] font-medium text-slate-500 mb-1">Preço</label>
              <div className="flex items-center gap-2">
                <input
                  type="number"
                  step="0.01"
                  min="0"
                  value={isSobConsulta ? '' : (produto.preco || '')}
                  onChange={e => update('preco', e.target.value === '' ? null : parseFloat(e.target.value))}
                  disabled={isSobConsulta}
                  placeholder={isSobConsulta ? 'Sob consulta' : 'R$ 0,00'}
                  className="flex-1 px-3 py-2 rounded-lg border border-slate-200 text-sm focus:border-rose-500 outline-none disabled:bg-slate-50 disabled:text-slate-400"
                />
              </div>
              <label className="flex items-center gap-2 mt-1.5 cursor-pointer">
                <input
                  type="checkbox"
                  checked={isSobConsulta}
                  onChange={toggleSobConsulta}
                  className="accent-rose-500"
                />
                <span className="text-[11px] text-slate-500">Sob consulta</span>
              </label>
            </div>
            <div>
              <label className="block text-[11px] font-medium text-slate-500 mb-1">Categoria</label>
              <select
                value={produto.categoria || ''}
                onChange={e => update('categoria', e.target.value)}
                className="w-full px-3 py-2 rounded-lg border border-slate-200 text-sm focus:border-rose-500 outline-none bg-white"
              >
                <option value="">Selecione...</option>
                {CATEGORIAS.map(c => (
                  <option key={c.value} value={c.value}>{c.label}</option>
                ))}
              </select>
            </div>
          </div>

          <div>
            <label className="block text-[11px] font-medium text-slate-500 mb-1">Observações</label>
            <input
              type="text"
              value={produto.observacoes || ''}
              onChange={e => update('observacoes', e.target.value)}
              placeholder="Ex: Consulte disponibilidade de tamanhos"
              className="w-full px-3 py-2 rounded-lg border border-slate-200 text-sm focus:border-rose-500 outline-none"
            />
          </div>

          <div className="pt-2 border-t border-slate-100 flex justify-end">
            <button
              type="button"
              onClick={onRemove}
              className="text-xs text-red-400 hover:text-red-600 hover:bg-red-50 px-3 py-1.5 rounded-lg transition"
            >
              Remover produto
            </button>
          </div>
        </div>
      )}
    </div>
  )
}

// ===== Main Roupas Editor =====
export default function RoupasEditor() {
  const { slug } = useParams()
  const [products, setProducts] = useState([])
  const [loading, setLoading] = useState(true)
  const [saving, setSaving] = useState(false)
  const [saved, setSaved] = useState(false)
  const [clientName, setClientName] = useState('')

  useEffect(() => {
    async function load() {
      try {
        const restSnap = await getDoc(doc(db, 'restaurants', slug))
        if (restSnap.exists()) setClientName(restSnap.data().name)

        const snap = await getDoc(doc(db, 'restaurants', slug, 'data', 'roupas'))
        if (snap.exists() && snap.data().content) {
          setProducts(snap.data().content)
        }
      } catch (err) {
        console.error('Erro ao carregar produtos:', err)
      }
      setLoading(false)
    }
    load()
  }, [slug])

  function updateProduct(index, newProduct) {
    const updated = [...products]
    updated[index] = newProduct
    setProducts(updated)
    setSaved(false)
  }

  function removeProduct(index) {
    const p = products[index]
    if (!confirm(`Remover "${p.nome || 'este produto'}"?`)) return
    setProducts(products.filter((_, i) => i !== index))
    setSaved(false)
  }

  function addProduct() {
    setProducts([...products, {
      id: generateId(),
      nome: '',
      descricao: '',
      preco: null,
      categoria: '',
      destaque: false,
      observacoes: '',
      images: []
    }])
    setSaved(false)
  }

  async function handleSave() {
    setSaving(true)
    try {
      const content = products.map(p => ({
        ...p,
        images: (p.images || []).map(img =>
          typeof img === 'string' ? { url: img, storagePath: '' } : img
        )
      }))

      await setDoc(doc(db, 'restaurants', slug, 'data', 'roupas'), {
        content,
        updatedAt: new Date().toISOString()
      })
      setSaved(true)
      setTimeout(() => setSaved(false), 3000)
    } catch (err) {
      alert('Erro ao salvar: ' + err.message)
    }
    setSaving(false)
  }

  if (loading) {
    return (
      <div className="flex justify-center py-20">
        <div className="animate-spin rounded-full h-10 w-10 border-b-2 border-rose-500"></div>
      </div>
    )
  }

  return (
    <div>
      <div className="flex items-center gap-3 mb-6">
        <Link to="/" className="text-slate-400 hover:text-slate-600">
          <svg className="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
            <path strokeLinecap="round" strokeLinejoin="round" d="M15 19l-7-7 7-7" />
          </svg>
        </Link>
        <div>
          <h1 className="text-xl font-bold text-slate-800">Catálogo de Roupas</h1>
          <p className="text-xs text-slate-400">{clientName}</p>
        </div>
        <div className="ml-auto flex gap-2 items-center">
          {saved && <span className="text-xs text-green-600">Salvo!</span>}
          <button
            onClick={handleSave}
            disabled={saving}
            className="px-5 py-2 bg-rose-500 hover:bg-rose-600 text-white text-sm font-semibold rounded-xl transition disabled:opacity-50"
          >
            {saving ? 'Salvando...' : 'Salvar'}
          </button>
        </div>
      </div>

      <div className="flex items-center justify-between mb-4">
        <p className="text-sm text-slate-500">
          {products.length} {products.length === 1 ? 'produto' : 'produtos'}
        </p>
        <button
          onClick={addProduct}
          className="text-sm bg-rose-50 hover:bg-rose-100 text-rose-700 px-4 py-2 rounded-xl transition font-medium"
        >
          + Novo produto
        </button>
      </div>

      {products.length === 0 ? (
        <div className="text-center py-16 text-slate-400">
          <p className="text-lg mb-2">Nenhum produto cadastrado</p>
          <p className="text-sm">Clique em "+ Novo produto" para começar</p>
        </div>
      ) : (
        <div className="space-y-3">
          {products.map((p, i) => (
            <ProductCard
              key={p.id}
              produto={p}
              onChange={(updated) => updateProduct(i, updated)}
              onRemove={() => removeProduct(i)}
              slug={slug}
            />
          ))}
        </div>
      )}
    </div>
  )
}
```

- [ ] **Step 2: Verify build**

```bash
cd C:\dev\cardapio-admin && npm run build
```

Expected: Build succeeds (RoupasEditor is created but not yet routed — verified next task).

- [ ] **Step 3: Commit**

```bash
cd C:\dev\cardapio-admin
git add src/pages/RoupasEditor.jsx
git commit -m "feat: create RoupasEditor page for clothing catalog management"
```

---

## Task 4: cardapio-admin — Add route for RoupasEditor

**Files:**
- Modify: `C:\dev\cardapio-admin\src\App.jsx`

- [ ] **Step 1: Import RoupasEditor**

Add after line 8 (`import BusinessInfoEditor from './pages/BusinessInfoEditor'`):

```jsx
import RoupasEditor from './pages/RoupasEditor'
```

- [ ] **Step 2: Add route**

Add after line 24 (`<Route path="/restaurante/:slug/info" element={<BusinessInfoEditor />} />`):

```jsx
          <Route path="/restaurante/:slug/roupas" element={<RoupasEditor />} />
```

- [ ] **Step 3: Verify build**

```bash
cd C:\dev\cardapio-admin && npm run build
```

Expected: Build succeeds with no errors.

- [ ] **Step 4: Commit**

```bash
cd C:\dev\cardapio-admin
git add src/App.jsx
git commit -m "feat: add route for RoupasEditor"
```

---

## Task 5: Seed script — Insert 33 initial products into Firestore

**Files:**
- Create: `C:\dev\cardapio-admin\seed-imperium.js`

This script uses the Firebase Admin SDK via the existing client-side Firebase config to seed products. It runs with Node.js using the Firebase client SDK (same approach the app uses).

- [ ] **Step 1: Create seed-imperium.js**

Create `C:\dev\cardapio-admin\seed-imperium.js`:

```javascript
import { initializeApp } from 'firebase/app'
import { getFirestore, doc, setDoc } from 'firebase/firestore'

const firebaseConfig = {
  apiKey: "AIzaSyD7ImWnSeSb3DTuyXTnS55gRsqkBZZv5q8",
  authDomain: "clientes-admin-a2258.firebaseapp.com",
  projectId: "clientes-admin-a2258",
  storageBucket: "clientes-admin-a2258.firebasestorage.app",
  messagingSenderId: "598293541730",
  appId: "1:598293541730:web:bdee1e314b46e5fa9ff23b"
}

const app = initializeApp(firebaseConfig)
const db = getFirestore(app)

const SLUG = 'imperium-moda-social'

const products = [
  { id: 'camisa-social-azul-marinho', nome: 'Camisa Social Azul Marinho levemente Acetinada', descricao: 'Super fácil de passar, não amassa durante o uso, sensação de frescor, antialérgica, toque macio, colarinho estruturado, não desbota, não encolhe/alarga pós lavagem. QUALIDADE SUPER PREMIUM.', preco: 229.90, categoria: 'camisas-sociais', destaque: false, observacoes: '', images: [] },
  { id: 'camisa-social-slim-branca-bambu', nome: 'Camisa Social Slim Branca (Fibra Natural de Bambu)', descricao: 'Tecido fresco, não amarrota durante o uso e super fácil de passar.', preco: 229.90, categoria: 'camisas-sociais', destaque: false, observacoes: '', images: [] },
  { id: 'camisa-social-branca-passa-facil', nome: 'Camisa Social sem bolso Branca (Passa fácil e não Amassa)', descricao: 'Fibra Natural de Bambu com Elástano. Tecnologia passa fácil, toque macio, sensação térmica agradável, antialérgica, manga longa s/ bolso, colarinho italiano (ideal p/ gravatas).', preco: 229.90, categoria: 'camisas-sociais', destaque: false, observacoes: 'Consulte disponibilidade de cores', images: [] },
  { id: 'camisa-social-slim-vivacci', nome: 'Camisas Social Slim Fit Vivacci (passa fácil e não amarrota)', descricao: 'Levemente acetinada ou fibra natural de bambu. Super fácil de passar, tecido leve e respirável, toque macio, antialérgica, caimento perfeito, colarinho estruturado, não desbota, não encolhe/alarga. Qualidade Super Premium.', preco: 229.90, categoria: 'camisas-sociais', destaque: false, observacoes: 'Consultar disponibilidade', images: [] },
  { id: 'cinto-dupla-face', nome: 'Cinto dupla face (Preto/Marrom)', descricao: 'Super versátil e elegante, qualidade e acabamento premium. Material: Sintético.', preco: 49.90, categoria: 'acessorios', destaque: false, observacoes: '', images: [] },
  { id: 'cintos-trava-automatica', nome: 'Cintos trava automática Premium', descricao: 'Trava automática ajustável, super elegante e prático, qualidade e acabamento perfeito.', preco: 129.90, categoria: 'acessorios', destaque: false, observacoes: '', images: [] },
  { id: 'costume-two-way-elastano', nome: 'Costume Two Way com elastano', descricao: 'Costume Two Way com elastano.', preco: 599.90, categoria: 'ternos-costumes', destaque: false, observacoes: 'Consulte disponibilidade de tamanhos e cores', images: [] },
  { id: 'bermudas-alfaiataria', nome: 'Bermudas alfaiataria', descricao: 'Diversas opções de cores e tecidos.', preco: 169.90, categoria: 'calcas-bermudas', destaque: false, observacoes: 'Consulte disponibilidade de cores e tamanhos', images: [] },
  { id: 'calcas-jeans', nome: 'Calças Jeans (Disponível do 38 ao 48)', descricao: 'Calças Jeans.', preco: 179.90, categoria: 'calcas-bermudas', destaque: false, observacoes: 'Consulte disponibilidade', images: [] },
  { id: 'camisa-social-elastic-sibra', nome: 'Camisa social tecnológica Elastic Sibra', descricao: 'Não precisa passar, não amarrota durante o uso, ultra elástica, antialérgica, antiodor, toque macio, tecido natural e ecológico, respirável, leve e fresquinha.', preco: 249.90, categoria: 'camisas-sociais', destaque: false, observacoes: '', images: [] },
  { id: 'jaqueta-sarja-premium', nome: 'Jaqueta Sarja Premium', descricao: 'Jaqueta em Sarja. Tamanho P ao GG. Cores: Verde militar, Preto. Composição: Algodão com elastano.', preco: null, categoria: 'jaquetas-sueteres', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'calcas-alfaiataria-regulagem', nome: 'Calças Alfaiataria com regulagem', descricao: 'Calças alfaiataria em malha com regulagem. Caimento impecável, conforto inigualável, praticidade, elegância e autoridade. Enviamos para todo Brasil.', preco: null, categoria: 'calcas-bermudas', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'tshirts-basicas', nome: 'T-shirts básicas diversas cores', descricao: 'Camisetas básicas em várias opções de cores, caimento sensacional e conforto extremo.', preco: null, categoria: 'camisetas-polos', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'sueter-premium', nome: 'Suéter Premium', descricao: 'Toque macio, caimento perfeito. Tamanhos: P ao EG. Cores: Preto, Cinza, Marinho, Marrom caramelo. Composição: Modal/Poliéster/Nylon.', preco: null, categoria: 'jaquetas-sueteres', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'perfumes-pronta-entrega', nome: 'Perfumes a pronta entrega', descricao: 'Mesma linhagem olfativa dos importados, muita fixação e fragrâncias irresistíveis.', preco: 159.90, categoria: 'perfumes', destaque: false, observacoes: '', images: [] },
  { id: 'camisas-polo-vivacci', nome: 'Camisas Polo Vivacci P ao G1', descricao: 'Toque macio, confortável, caimento perfeito, resistente a rugas, fácil de passar. Disponível do P ao GG.', preco: 149.90, categoria: 'camisetas-polos', destaque: false, observacoes: 'Consultar disponibilidade de cores e tamanhos', images: [] },
  { id: 'ternos-poliviscose-slim', nome: 'Ternos Poliviscose Slim Italiano', descricao: 'Diversos modelos.', preco: 799.90, categoria: 'ternos-costumes', destaque: false, observacoes: 'Consulte lançamentos e disponibilidade', images: [] },
  { id: 'calcas-alfaiataria-38-52', nome: 'Calças alfaiataria (disponível do 38 ao 52)', descricao: 'Calças alfaiataria.', preco: null, categoria: 'calcas-bermudas', destaque: false, observacoes: 'Consulte disponibilidade', images: [] },
  { id: 'gravatas-variedade', nome: 'Variedade em Gravatas', descricao: 'Vários modelos para todas as ocasiões: eventos religiosos, casamentos, formaturas, apresentações e trabalhos.', preco: null, categoria: 'acessorios', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'looks-combinacoes', nome: 'Looks, Combinações e Conjuntos', descricao: 'Opções de looks, combinações e conjuntos disponíveis.', preco: null, categoria: 'kits-combinacoes', destaque: false, observacoes: 'Consulte disponibilidade', images: [] },
  { id: 'cintos-varios-modelos', nome: 'Cintos em vários modelos, tamanhos e cores', descricao: 'Opções: Couro legítimo, Couro legítimo com elástico, Trava automática em couro legítimo, Dupla face sintético, Elástico fivela de trava.', preco: null, categoria: 'acessorios', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'camisetas-polos-diversos', nome: 'Camisetas Polos diversos modelos, cores e tamanhos', descricao: 'Diversos modelos, tamanhos e cores.', preco: null, categoria: 'camisetas-polos', destaque: false, observacoes: 'Consulte disponibilidade', images: [] },
  { id: 'ternos-costumes-variedade', nome: 'Ternos e costumes (Variedade em cores e tecidos)', descricao: 'Variedade em cores e tecidos.', preco: null, categoria: 'ternos-costumes', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'gravatas-slim-xadrez', nome: 'Gravatas Slim Xadrez 1.200 Fios', descricao: 'Muitos modelos disponíveis.', preco: 29.90, categoria: 'acessorios', destaque: false, observacoes: 'Consultar disponibilidade', images: [] },
  { id: 'cinto-couro-fasolo', nome: 'Cinto Couro Nobre Fasolo', descricao: 'Marca nacional, qualidade garantida. Diversos tamanhos disponíveis.', preco: 99.90, categoria: 'acessorios', destaque: false, observacoes: 'Consultar disponibilidade dos tamanhos', images: [] },
  { id: 'kit-calca-camisa-polo', nome: 'Kit Perfeito Calça Alfaiataria + Camisa Polo Importada Vivacci', descricao: 'Calça alfaiataria + camisa polo importada Vivacci. Modelos exclusivos da marca.', preco: null, categoria: 'kits-combinacoes', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'ternos-slim-microfibra', nome: 'Ternos Slim Microfibra Italiano', descricao: 'Diversos modelos.', preco: 499.90, categoria: 'ternos-costumes', destaque: false, observacoes: 'Consultar disponibilidade', images: [] },
  { id: 'ternos-fio-indiano', nome: 'Ternos Fio Indiano Slim Italiano', descricao: 'Diversos modelos.', preco: 599.90, categoria: 'ternos-costumes', destaque: false, observacoes: 'Consultar disponibilidade', images: [] },
  { id: 'calcas-sarjas-alfaiataria', nome: 'Calças Sarjas Alfaiataria', descricao: 'Algodão com elástano. Bolsos dianteiro modelo faca, bolsos traseiro embutidos, modelagem slim.', preco: null, categoria: 'calcas-bermudas', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'gravatas-importadas-slim', nome: 'Gravatas Importadas Slim (Vários modelos)', descricao: 'Estampas variadas.', preco: null, categoria: 'acessorios', destaque: false, observacoes: 'Consulte disponibilidade', images: [] },
  { id: 'kit-camisa-calca-esporte', nome: 'Kit 2 Camisa Social + Calça Esporte fino', descricao: 'Camisa Social Slim Fit passa fácil importada + Calça Alfaiataria esporte fino.', preco: null, categoria: 'kits-combinacoes', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'kit-camisa-gravata-prendedor', nome: 'Kit 1 Camisa social + Gravata + Prendedor de Gravata', descricao: 'Camisa Social Slim Fit importada passa fácil + Gravata importada Slim + Prendedor de gravata Slim.', preco: null, categoria: 'kits-combinacoes', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'cinto-elastico-trava', nome: 'Cinto Elástico com trava', descricao: 'Cinto de elástico com trava (conforto e eficiência).', preco: null, categoria: 'acessorios', destaque: false, observacoes: 'Consultar preço', images: [] },
]

const businessInfo = {
  name: 'Imperium Moda Social Masculina',
  city: 'Taquaritinga',
  slogan: 'Elegância masculina para quem quer presença, estilo e sofisticação.',
  tagline: 'Moda Social Masculina Premium',
  whatsapp: '(16) 99161-1681',
  whatsappNumber: '5516991611681',
  phone: '(16) 99161-1681',
  address: 'Rua dos Domingues, 534',
  neighborhood: 'Centro',
  cityState: 'Taquaritinga - SP',
  cep: '15900-023',
  hours: { weekdays: '09:00 - 18:00', saturday: '09:00 - 13:00' },
  instagram: '@imperium.modasocial',
  facebook: '',
  googleMapsEmbed: '',
  googleMapsLink: ''
}

async function seed() {
  console.log('Creating restaurant document...')
  await setDoc(doc(db, 'restaurants', SLUG), {
    name: 'Imperium Moda Social Masculina',
    slug: SLUG,
    type: 'roupas',
    createdAt: new Date().toISOString()
  })

  console.log(`Seeding ${products.length} products...`)
  await setDoc(doc(db, 'restaurants', SLUG, 'data', 'roupas'), {
    content: products,
    updatedAt: new Date().toISOString()
  })

  console.log('Seeding business info...')
  await setDoc(doc(db, 'restaurants', SLUG, 'data', 'businessInfo'), {
    content: businessInfo,
    updatedAt: new Date().toISOString()
  })

  console.log('Done! Seeded:')
  console.log(`- ${products.length} products`)
  console.log('- 1 businessInfo document')
  console.log(`- Slug: ${SLUG}`)
  process.exit(0)
}

seed().catch(err => { console.error('Seed failed:', err); process.exit(1) })
```

- [ ] **Step 2: Run the seed script**

```bash
cd C:\dev\cardapio-admin && node seed-imperium.js
```

Expected output:
```
Creating restaurant document...
Seeding 33 products...
Seeding business info...
Done! Seeded:
- 33 products
- 1 businessInfo document
- Slug: imperium-moda-social
```

- [ ] **Step 3: Commit**

```bash
cd C:\dev\cardapio-admin
git add seed-imperium.js
git commit -m "feat: add seed script for Imperium Moda Social products"
```

---

## Task 6: Frontend — Landing page (index.html)

**Files:**
- Create: `C:\dev\prototipos\imperium moda social masculina\index.html`

This is the core deliverable. The landing page follows the Marieta pattern: all CSS inline in `<style>`, JS inline at the bottom, with the Firestore REST API fetch for businessInfo.

- [ ] **Step 1: Create index.html**

Create `C:\dev\prototipos\imperium moda social masculina\index.html` with the complete landing page.

Structure requirements (from spec):
- HTML5 with semantic tags, lang="pt-BR"
- Google Fonts: Playfair Display (400, 700) + Inter (300, 400, 600)
- CSS variables for the color palette (see spec: Paleta section)
- All 11 sections from the spec (Header through Footer)
- WhatsApp floating button
- Scroll reveal animations (IntersectionObserver)
- Navbar scroll behavior (transparent → solid)
- Mobile hamburger menu
- Schema.org JSON-LD (LocalBusiness)
- SEO meta tags from spec
- Firestore REST API fetch for businessInfo (same pattern as Marieta)
- Fallback businessInfo hardcoded in JS

Key CSS variables:
```css
:root {
  --black: #0a0a0a;
  --graphite: #1a1a1a;
  --card-bg: #111111;
  --white: #ffffff;
  --gray: #999999;
  --gold: #d4af37;
  --gold-dark: #b8960c;
  --gold-light: #c9a227;
  --whatsapp: #25D366;
  --font-display: 'Playfair Display', Georgia, serif;
  --font-body: 'Inter', sans-serif;
  --ease-out: cubic-bezier(0.16, 1, 0.3, 1);
}
```

Key Firestore config:
```javascript
var RESTAURANT_SLUG = 'imperium-moda-social';
var FIRESTORE_PROJECT = 'clientes-admin-a2258';
var FIRESTORE_BASE = 'https://firestore.googleapis.com/v1/projects/' + FIRESTORE_PROJECT + '/databases/(default)/documents/restaurants/' + RESTAURANT_SLUG + '/data/';
var isLocal = location.protocol === 'file:';
```

The HTML must include all section content from the spec — all copy text, all CTAs, all depoimentos, all diferenciais items. Refer to spec sections 1-11 for exact text content.

WhatsApp link format: `https://wa.me/5516991611681?text=Olá! Gostaria de conhecer as peças da Imperium.`

Menu links: Use `data-section` attributes matching section IDs for smooth scroll navigation. "Catálogo" links to `catalogo.html`.

- [ ] **Step 2: Open in browser and verify**

```bash
start "" "C:\dev\prototipos\imperium moda social masculina\index.html"
```

Verify:
- All 10 sections render correctly
- Navbar becomes solid on scroll
- Mobile hamburger menu works (resize browser)
- WhatsApp floating button visible
- Smooth scroll navigation works
- Scroll reveal animations trigger
- All text content matches spec

- [ ] **Step 3: Commit**

```bash
cd "C:\dev\prototipos\imperium moda social masculina"
git add index.html
git commit -m "feat: create landing page with all sections, SEO, and Firestore integration"
```

---

## Task 7: Frontend — Fallback data file (roupas.js)

**Files:**
- Create: `C:\dev\prototipos\imperium moda social masculina\roupas.js`

- [ ] **Step 1: Create roupas.js**

This is the fallback data file, equivalent to Marieta's `cardapio.js`. Contains all 33 products hardcoded for offline/file:// use.

```javascript
var roupasData = [
  { id: 'camisa-social-azul-marinho', nome: 'Camisa Social Azul Marinho levemente Acetinada', descricao: 'Super fácil de passar, não amassa durante o uso, sensação de frescor, antialérgica, toque macio, colarinho estruturado, não desbota, não encolhe/alarga pós lavagem. QUALIDADE SUPER PREMIUM.', preco: 229.90, categoria: 'camisas-sociais', destaque: false, observacoes: '', images: [] },
  { id: 'camisa-social-slim-branca-bambu', nome: 'Camisa Social Slim Branca (Fibra Natural de Bambu)', descricao: 'Tecido fresco, não amarrota durante o uso e super fácil de passar.', preco: 229.90, categoria: 'camisas-sociais', destaque: false, observacoes: '', images: [] },
  { id: 'camisa-social-branca-passa-facil', nome: 'Camisa Social sem bolso Branca (Passa fácil e não Amassa)', descricao: 'Fibra Natural de Bambu com Elástano. Tecnologia passa fácil, toque macio, sensação térmica agradável, antialérgica, manga longa s/ bolso, colarinho italiano (ideal p/ gravatas).', preco: 229.90, categoria: 'camisas-sociais', destaque: false, observacoes: 'Consulte disponibilidade de cores', images: [] },
  { id: 'camisa-social-slim-vivacci', nome: 'Camisas Social Slim Fit Vivacci (passa fácil e não amarrota)', descricao: 'Levemente acetinada ou fibra natural de bambu. Super fácil de passar, tecido leve e respirável, toque macio, antialérgica, caimento perfeito, colarinho estruturado, não desbota, não encolhe/alarga. Qualidade Super Premium.', preco: 229.90, categoria: 'camisas-sociais', destaque: false, observacoes: 'Consultar disponibilidade', images: [] },
  { id: 'cinto-dupla-face', nome: 'Cinto dupla face (Preto/Marrom)', descricao: 'Super versátil e elegante, qualidade e acabamento premium. Material: Sintético.', preco: 49.90, categoria: 'acessorios', destaque: false, observacoes: '', images: [] },
  { id: 'cintos-trava-automatica', nome: 'Cintos trava automática Premium', descricao: 'Trava automática ajustável, super elegante e prático, qualidade e acabamento perfeito.', preco: 129.90, categoria: 'acessorios', destaque: false, observacoes: '', images: [] },
  { id: 'costume-two-way-elastano', nome: 'Costume Two Way com elastano', descricao: 'Costume Two Way com elastano.', preco: 599.90, categoria: 'ternos-costumes', destaque: false, observacoes: 'Consulte disponibilidade de tamanhos e cores', images: [] },
  { id: 'bermudas-alfaiataria', nome: 'Bermudas alfaiataria', descricao: 'Diversas opções de cores e tecidos.', preco: 169.90, categoria: 'calcas-bermudas', destaque: false, observacoes: 'Consulte disponibilidade de cores e tamanhos', images: [] },
  { id: 'calcas-jeans', nome: 'Calças Jeans (Disponível do 38 ao 48)', descricao: 'Calças Jeans.', preco: 179.90, categoria: 'calcas-bermudas', destaque: false, observacoes: 'Consulte disponibilidade', images: [] },
  { id: 'camisa-social-elastic-sibra', nome: 'Camisa social tecnológica Elastic Sibra', descricao: 'Não precisa passar, não amarrota durante o uso, ultra elástica, antialérgica, antiodor, toque macio, tecido natural e ecológico, respirável, leve e fresquinha.', preco: 249.90, categoria: 'camisas-sociais', destaque: false, observacoes: '', images: [] },
  { id: 'jaqueta-sarja-premium', nome: 'Jaqueta Sarja Premium', descricao: 'Jaqueta em Sarja. Tamanho P ao GG. Cores: Verde militar, Preto. Composição: Algodão com elastano.', preco: null, categoria: 'jaquetas-sueteres', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'calcas-alfaiataria-regulagem', nome: 'Calças Alfaiataria com regulagem', descricao: 'Calças alfaiataria em malha com regulagem. Caimento impecável, conforto inigualável, praticidade, elegância e autoridade. Enviamos para todo Brasil.', preco: null, categoria: 'calcas-bermudas', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'tshirts-basicas', nome: 'T-shirts básicas diversas cores', descricao: 'Camisetas básicas em várias opções de cores, caimento sensacional e conforto extremo.', preco: null, categoria: 'camisetas-polos', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'sueter-premium', nome: 'Suéter Premium', descricao: 'Toque macio, caimento perfeito. Tamanhos: P ao EG. Cores: Preto, Cinza, Marinho, Marrom caramelo. Composição: Modal/Poliéster/Nylon.', preco: null, categoria: 'jaquetas-sueteres', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'perfumes-pronta-entrega', nome: 'Perfumes a pronta entrega', descricao: 'Mesma linhagem olfativa dos importados, muita fixação e fragrâncias irresistíveis.', preco: 159.90, categoria: 'perfumes', destaque: false, observacoes: '', images: [] },
  { id: 'camisas-polo-vivacci', nome: 'Camisas Polo Vivacci P ao G1', descricao: 'Toque macio, confortável, caimento perfeito, resistente a rugas, fácil de passar. Disponível do P ao GG.', preco: 149.90, categoria: 'camisetas-polos', destaque: false, observacoes: 'Consultar disponibilidade de cores e tamanhos', images: [] },
  { id: 'ternos-poliviscose-slim', nome: 'Ternos Poliviscose Slim Italiano', descricao: 'Diversos modelos.', preco: 799.90, categoria: 'ternos-costumes', destaque: false, observacoes: 'Consulte lançamentos e disponibilidade', images: [] },
  { id: 'calcas-alfaiataria-38-52', nome: 'Calças alfaiataria (disponível do 38 ao 52)', descricao: 'Calças alfaiataria.', preco: null, categoria: 'calcas-bermudas', destaque: false, observacoes: 'Consulte disponibilidade', images: [] },
  { id: 'gravatas-variedade', nome: 'Variedade em Gravatas', descricao: 'Vários modelos para todas as ocasiões: eventos religiosos, casamentos, formaturas, apresentações e trabalhos.', preco: null, categoria: 'acessorios', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'looks-combinacoes', nome: 'Looks, Combinações e Conjuntos', descricao: 'Opções de looks, combinações e conjuntos disponíveis.', preco: null, categoria: 'kits-combinacoes', destaque: false, observacoes: 'Consulte disponibilidade', images: [] },
  { id: 'cintos-varios-modelos', nome: 'Cintos em vários modelos, tamanhos e cores', descricao: 'Opções: Couro legítimo, Couro legítimo com elástico, Trava automática em couro legítimo, Dupla face sintético, Elástico fivela de trava.', preco: null, categoria: 'acessorios', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'camisetas-polos-diversos', nome: 'Camisetas Polos diversos modelos, cores e tamanhos', descricao: 'Diversos modelos, tamanhos e cores.', preco: null, categoria: 'camisetas-polos', destaque: false, observacoes: 'Consulte disponibilidade', images: [] },
  { id: 'ternos-costumes-variedade', nome: 'Ternos e costumes (Variedade em cores e tecidos)', descricao: 'Variedade em cores e tecidos.', preco: null, categoria: 'ternos-costumes', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'gravatas-slim-xadrez', nome: 'Gravatas Slim Xadrez 1.200 Fios', descricao: 'Muitos modelos disponíveis.', preco: 29.90, categoria: 'acessorios', destaque: false, observacoes: 'Consultar disponibilidade', images: [] },
  { id: 'cinto-couro-fasolo', nome: 'Cinto Couro Nobre Fasolo', descricao: 'Marca nacional, qualidade garantida. Diversos tamanhos disponíveis.', preco: 99.90, categoria: 'acessorios', destaque: false, observacoes: 'Consultar disponibilidade dos tamanhos', images: [] },
  { id: 'kit-calca-camisa-polo', nome: 'Kit Perfeito Calça Alfaiataria + Camisa Polo Importada Vivacci', descricao: 'Calça alfaiataria + camisa polo importada Vivacci. Modelos exclusivos da marca.', preco: null, categoria: 'kits-combinacoes', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'ternos-slim-microfibra', nome: 'Ternos Slim Microfibra Italiano', descricao: 'Diversos modelos.', preco: 499.90, categoria: 'ternos-costumes', destaque: false, observacoes: 'Consultar disponibilidade', images: [] },
  { id: 'ternos-fio-indiano', nome: 'Ternos Fio Indiano Slim Italiano', descricao: 'Diversos modelos.', preco: 599.90, categoria: 'ternos-costumes', destaque: false, observacoes: 'Consultar disponibilidade', images: [] },
  { id: 'calcas-sarjas-alfaiataria', nome: 'Calças Sarjas Alfaiataria', descricao: 'Algodão com elástano. Bolsos dianteiro modelo faca, bolsos traseiro embutidos, modelagem slim.', preco: null, categoria: 'calcas-bermudas', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'gravatas-importadas-slim', nome: 'Gravatas Importadas Slim (Vários modelos)', descricao: 'Estampas variadas.', preco: null, categoria: 'acessorios', destaque: false, observacoes: 'Consulte disponibilidade', images: [] },
  { id: 'kit-camisa-calca-esporte', nome: 'Kit 2 Camisa Social + Calça Esporte fino', descricao: 'Camisa Social Slim Fit passa fácil importada + Calça Alfaiataria esporte fino.', preco: null, categoria: 'kits-combinacoes', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'kit-camisa-gravata-prendedor', nome: 'Kit 1 Camisa social + Gravata + Prendedor de Gravata', descricao: 'Camisa Social Slim Fit importada passa fácil + Gravata importada Slim + Prendedor de gravata Slim.', preco: null, categoria: 'kits-combinacoes', destaque: false, observacoes: 'Consultar preço', images: [] },
  { id: 'cinto-elastico-trava', nome: 'Cinto Elástico com trava', descricao: 'Cinto de elástico com trava (conforto e eficiência).', preco: null, categoria: 'acessorios', destaque: false, observacoes: 'Consultar preço', images: [] },
];
```

- [ ] **Step 2: Commit**

```bash
cd "C:\dev\prototipos\imperium moda social masculina"
git add roupas.js
git commit -m "feat: add fallback product data for offline use"
```

---

## Task 8: Frontend — Catalog page (catalogo.html)

**Files:**
- Create: `C:\dev\prototipos\imperium moda social masculina\catalogo.html`

- [ ] **Step 1: Create catalogo.html**

Create `C:\dev\prototipos\imperium moda social masculina\catalogo.html` with the complete catalog page.

Structure requirements (from spec — Catalog section):
- Same header/navbar as index.html (copy the HTML/CSS)
- Banner compacto (~200px) with "Catálogo" title + subtitle
- Category filter pills (8 categories + "Todos")
- Product grid (3 col desktop, 2 tablet, 1 mobile)
- Product modal with gallery, description, price, WhatsApp button
- Same footer as index.html
- Same WhatsApp floating button
- Firestore REST API fetch for `roupas` data
- Fallback to `roupasData` from `roupas.js`
- URL param `?cat=slug` for pre-filtered view

Key JS functions to implement:
```javascript
// Fetch products from Firestore (same parseFirestoreValue pattern as Marieta)
function fetchFirestore(docName) { ... }

// Render product grid
function renderProducts(products, filter) { ... }

// Open product modal
function openModal(product) { ... }

// Close modal
function closeModal() { ... }

// Filter by category
function filterByCategory(cat) { ... }

// Format price
function formatPrice(preco) {
  if (preco === null || preco === undefined) return 'Sob consulta';
  return 'R$ ' + Number(preco).toFixed(2).replace('.', ',');
}
```

Modal WhatsApp link: `https://wa.me/5516991611681?text=Olá! Vi o produto "${produto.nome}" no site e gostaria de mais informações.`

Category labels mapping (for filter pills):
```javascript
var CATEGORIAS = [
  { slug: 'camisas-sociais', label: 'Camisas Sociais' },
  { slug: 'camisetas-polos', label: 'Camisetas & Polos' },
  { slug: 'calcas-bermudas', label: 'Calças & Bermudas' },
  { slug: 'ternos-costumes', label: 'Ternos & Costumes' },
  { slug: 'acessorios', label: 'Acessórios' },
  { slug: 'kits-combinacoes', label: 'Kits & Combinações' },
  { slug: 'jaquetas-sueteres', label: 'Jaquetas & Suéteres' },
  { slug: 'perfumes', label: 'Perfumes' },
];
```

- [ ] **Step 2: Open in browser and verify**

```bash
start "" "C:\dev\prototipos\imperium moda social masculina\catalogo.html"
```

Verify:
- Banner renders with title
- Category filter pills are visible and clickable
- All 33 products render in grid (using fallback data when opened via file://)
- Click on product opens modal with correct data
- Modal shows product name, description, price/sob consulta, observations
- WhatsApp button in modal has correct pre-filled message
- Modal closes on X click, outside click, and ESC
- Category filter works (shows/hides products)
- URL param `?cat=camisas-sociais` pre-selects filter
- Responsive: 3 cols → 2 cols → 1 col
- Navigation back to index.html works

- [ ] **Step 3: Commit**

```bash
cd "C:\dev\prototipos\imperium moda social masculina"
git add catalogo.html
git commit -m "feat: create catalog page with product grid, filters, and product modal"
```

---

## Task 9: Final verification and polish

- [ ] **Step 1: Verify Firestore data is accessible**

Open `catalogo.html` via a local HTTP server (not file://) to test Firestore integration:

```bash
cd "C:\dev\prototipos\imperium moda social masculina" && npx serve .
```

Open `http://localhost:3000` in browser. Verify:
- Products load from Firestore (not fallback)
- businessInfo loads on landing page (contact section populates)

- [ ] **Step 2: Verify admin builds and runs**

```bash
cd C:\dev\cardapio-admin && npm run build
```

Expected: Build succeeds.

```bash
cd C:\dev\cardapio-admin && npm run dev
```

Verify in browser:
- Dashboard shows "Imperium Moda Social Masculina" with "👔 Roupas" badge
- "Catálogo" button navigates to RoupasEditor
- RoupasEditor shows 33 products loaded from Firestore
- Products can be edited (name, description, price, category)
- "Sob consulta" checkbox works
- Save button saves to Firestore
- "Informações" button navigates to BusinessInfoEditor

- [ ] **Step 3: Commit any final fixes**

```bash
cd "C:\dev\prototipos\imperium moda social masculina"
git add -A
git commit -m "fix: final polish and fixes"
```

```bash
cd C:\dev\cardapio-admin
git add -A
git commit -m "fix: final polish and fixes"
```
