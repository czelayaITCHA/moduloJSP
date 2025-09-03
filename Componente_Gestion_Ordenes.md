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
## 3.- Crear *ordenService.js* dentro de la carpeta services del proyecto frontend (orders-app)
En este archivo programar funciones exportables para la gestión de las ordenes

```JavaScript
import axios from "axios";
import { urlBase } from "../../utils/config";


const buildHeaders = (token) => ({
  headers: {
    Authorization: token ? `Bearer ${token}` : "",
    "Content-Type": "application/json",
  },
});

export const getMenus = async (token) => {
  const res = await axios.get(`${urlBase}/menus`, buildHeaders(token));
  return res.data;
};

export const getMesas = async (token) => {
  const res = await axios.get(`${urlBase}/mesas`, buildHeaders(token));
  return res.data;
};

export const getClientes = async (token) => {
  const res = await axios.get(`${urlBase}/clientes`, buildHeaders(token));
  return res.data;
};

export const createOrden = async (dto, token) => {
  const res = await axios.post(`${url}/ordenes`, dto, buildHeaders(token));
  return res.data;
};

```

## 4.- Crear componentes de las vistas de acuerdo al rol del usuario
En la carpeta ordenes vamos a crear los archivos de los diferentes componentes para gestionar las ordenes, por el momento serán solo archivos con algun texto, luego se programará lógica real de cada componente

### 4.1 Crear componente *MenuCard.jsx* 
Este componente contendrá tarjeta del producto o menú con información como imagen, nombre, precio, cantidad, botón agregar

```JavaScript

import React, { useState } from "react";
import { FaPlus, FaMinus } from "react-icons/fa";
import { IMAGES_URL } from "../../utils/config";

const MenuCard = ({ menu, onAdd }) => {
  const [cantidad, setCantidad] = useState(1);

  const inc = () => setCantidad((c) => Math.min(99, c + 1));
  const dec = () => setCantidad((c) => Math.max(1, c - 1));

  return (
    <div className="bg-white rounded-xl shadow-md overflow-hidden flex flex-col">
      <div className="h-40 w-full overflow-hidden">
        <img
          src={`${IMAGES_URL}${menu.urlImagen}` || "/images/no-image.png"}
          alt={menu.nombre}
          className="object-cover w-full h-full"
        />
      </div>

      <div className="p-4 flex-1 flex flex-col">
        <div className="flex justify-between items-start">
          <h3 className="text-lg font-semibold text-slate-800">{menu.nombre}</h3>
          <div className="text-blue-700 font-bold">${Number(menu.precioUnitario).toFixed(2)}</div>
        </div>

        <p className="text-sm text-slate-500 mt-2 line-clamp-2">{menu.descripcion}</p>

        <div className="mt-auto pt-3 flex items-center justify-between">
          <div className="flex items-center gap-2 border rounded-md px-2 py-1">
            <button
              onClick={dec}
              className="p-1 rounded hover:bg-slate-100 active:scale-95"
              aria-label="Disminuir cantidad"
              type="button"
            >
              <FaMinus />
            </button>
            <input
              type="number"
              value={cantidad}
              min={1}
              onChange={(e) => {
                const v = parseInt(e.target.value || "1", 10);
                setCantidad(isNaN(v) ? 1 : Math.max(1, Math.min(99, v)));
              }}
              className="w-12 text-center text-sm outline-none"
            />
            <button
              onClick={inc}
              className="p-1 rounded hover:bg-slate-100 active:scale-95"
              aria-label="Aumentar cantidad"
              type="button"
            >
              <FaPlus />
            </button>
          </div>

          <button
            onClick={() => onAdd(menu, cantidad)}
            className="bg-blue-700 text-white px-3 py-1 rounded-lg shadow-sm hover:bg-blue-800"
            type="button"
          >
            Añadir
          </button>
        </div>
      </div>
    </div>
  );
};

export default MenuCard;

```




