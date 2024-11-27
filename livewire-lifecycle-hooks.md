# Livewire Lifecycle Hooks

Livewire provides a set of lifecycle hooks that are similar in concept to React's lifecycle methods. These hooks allow you to control various stages of a Livewire component's lifecycle.

---

## **Lifecycle Hooks in Livewire**

### **1. `mount()`**
- Called **once** when the Livewire component is initialized.
- Typically used to set up initial state or data.

#### **Example:**
```php
public function mount($id)
{
    $this->user = User::find($id);
}
```

---

### **2. `render()`**
- Called every time the component is rendered.
- Must return the component's view.

#### **Example:**
```php
public function render()
{
    return view('livewire.user-profile', [
        'posts' => Post::where('user_id', $this->user->id)->get()
    ]);
}
```

---

### **3. `updated()`**
- Called whenever a Livewire property is updated.
- Useful for additional logic when specific state changes.

#### **Example:**
```php
public $search;

public function updated($propertyName)
{
    $this->validateOnly($propertyName, [
        'search' => 'min:3',
    ]);
}
```

---

### **4. `updating()`**
- Called before a Livewire property is updated.
- Useful for preventing or manipulating data before it’s updated.

#### **Example:**
```php
public $search;

public function updating($propertyName, $value)
{
    if ($propertyName === 'search' && strlen($value) < 3) {
        $this->reset('search');
    }
}
```

---

### **5. `updated<PropertyName>()`**
- Called when a specific property is updated.

#### **Example:**
```php
public $name;

public function updatedName()
{
    $this->validate(['name' => 'required|min:3']);
}
```

---

### **6. `updating<PropertyName>()`**
- Called before a specific property is updated.

#### **Example:**
```php
public $name;

public function updatingName($value)
{
    if (strlen($value) > 50) {
        $this->reset('name');
    }
}
```

---

### **7. `hydrate()`**
- Called every time a Livewire component is rehydrated from the server after an AJAX request.
- Typically used to reset state or load additional data.

#### **Example:**
```php
public function hydrate()
{
    $this->user = auth()->user();
}
```

---

### **8. `dehydrate()`**
- Called every time a Livewire component is dehydrated and sent back to the client.
- Useful for cleaning up or modifying state before sending it to the client.

#### **Example:**
```php
public function dehydrate()
{
    unset($this->tempData);
}
```

---

### **9. `dehydrate<PropertyName>()`**
- Called when a specific property is dehydrated and sent to the client.

#### **Example:**
```php
public function dehydrateSearch()
{
    $this->search = trim($this->search);
}
```

---

### **10. `destroy()`**
- Called when a Livewire component is removed from the DOM.
- Similar to React's `componentWillUnmount()`.

#### **Example:**
```php
public function destroy()
{
    session()->forget('some-data');
}
```

---

### **11. Additional Lifecycle Hooks**
- **`boot()`**: Called once when the component is instantiated (similar to a constructor).
- **`booted()`**: Called after `boot()` and before the component is mounted.

---

## **Comparison: React vs Livewire Lifecycle Hooks**

| **React**                  | **Livewire**               | **Description**                                                                 |
|----------------------------|----------------------------|--------------------------------------------------------------------------------|
| `componentDidMount`        | `mount()`                 | Called when the component is first loaded.                                     |
| `componentDidUpdate`       | `updated()` / `updated<PropertyName>()` | Called when state or props are updated.                                        |
| `componentWillUnmount`     | `destroy()`               | Called when the component is removed from the DOM.                             |
| `render()`                 | `render()`                | Defines what is displayed in the view.                                         |
| `getDerivedStateFromProps` | `updating()` / `updating<PropertyName>()` | Manipulates data before it is updated.                                         |

---

## **When to Use Lifecycle Hooks in Livewire**

1. **`mount()`**: Use this to initialize data or receive parameters.
2. **`updated()`**: To validate or process data when state changes.
3. **`hydrate()`**: Use this to reset data after each AJAX request.
4. **`destroy()`**: Clean up resources when the component is removed.

---

## **Key Differences**
- Livewire lifecycle hooks run on the server-side and involve AJAX communication.
- React lifecycle methods operate entirely on the client-side.

---

This document provides a comprehensive guide to understanding and using Livewire's lifecycle hooks effectively. Let me know if you'd like further clarifications or examples!
