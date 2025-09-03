# Componente Gestión de Ordenes

## 1.- Crear carpeta de ordenes dentro components, en el proyecto Orders-app (proyecto react), tal como se muestra en la siguiente imágen

<img width="257" height="212" alt="image" src="https://github.com/user-attachments/assets/20cb943a-30cf-49d3-81ee-f0f9d2c6afb7" />

## 2.- Agregar o modificar opción de *Gestión Ordenes* en el Siderbar.jsx

```JavaScript
 {(user?.role === "ADMIN" || user?.role === "MESERO/A" || user?.role === "COCINERO" || user?.role === "CAJERO") && (
  <li>
    <Link
      to="/ordenes"
      className="p-4 hover:bg-blue-800 cursor-pointer flex items-center"
      onClick={handleMenuClick}
    >
      <FaBorderStyle className="mr-2" />
      Gestión Ordenes
    </Link>
  </li>
)}

```

