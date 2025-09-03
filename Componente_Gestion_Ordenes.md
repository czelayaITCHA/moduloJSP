# Componente Gestión de Ordenes
## Requisitos del backend para crear componente de gestión de ordenes en el frontend

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
    Authorization: `Bearer ${token}`,
    "Content-Type": "application/json",
  },
});

export const getMenus = async (token) => {
  const response = await axios.get(`${urlBase}menus`, buildHeaders(token));
  return response.data;
};

export const getMesas = async (token) => {
  const response = await axios.get(`${urlBase}mesas`, buildHeaders(token));
  return response.data;
};

export const getClientes = async (token) => {
  const response = await axios.get(`${urlBase}clientes`, buildHeaders(token));
  return response.data;
};

export const createOrden = async (dto, token) => {
  const response = await axios.post(`${urlBase}ordenes`, dto, buildHeaders(token));
  return response.data;
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
### 4.2.- Crear componente OrderCart.jsx
Panel lateral / drawer que muestra los items agregados, editar cantidad, eliminar, subtotal y botón para abrir confirmación. 

```JavaScript
import React from "react";
import { FaTrash } from "react-icons/fa";
import { IMAGES_URL } from "../../utils/config";

const formatCurrency = (v) => `$${Number(v).toFixed(2)}`;

const OrderCart = ({ items, onRemove, onUpdateQty, onOpenConfirm, total }) => {
  return (
    <aside className="bg-white rounded-xl shadow-lg p-4 w-full md:w-96">
      <h3 className="text-lg font-bold text-slate-800 mb-2">Orden en construcción</h3>

      {items.length === 0 ? (
        <div className="text-slate-500">No hay productos. Añade desde el menú.</div>
      ) : (
        <ul className="divide-y">
          {items.map((it) => (
            <li key={it.menu.id} className="py-3 flex items-center gap-3">
              <img
                src={`${IMAGES_URL}${it.menu.urlImagen}` || "/images/no-image.png"}
                alt={it.menu.nombre}
                className="w-12 h-12 object-cover rounded"
              />
              <div className="flex-1">
                <div className="flex justify-between items-center">
                  <div className="font-medium text-slate-800">{it.menu.nombre}</div>
                  <div className="text-slate-600 text-sm">{formatCurrency(it.precio)}</div>
                </div>
                <div className="mt-1 flex items-center gap-2">
                  <button
                    onClick={() => onUpdateQty(it.menu.id, Math.max(1, it.cantidad - 1))}
                    className="px-2 py-0.5 rounded border"
                    type="button"
                    aria-label="disminuir"
                  >
                    -
                  </button>
                  <input
                    type="number"
                    value={it.cantidad}
                    min={1}
                    onChange={(e) => onUpdateQty(it.menu.id, Math.max(1, Number(e.target.value || 1)))}
                    className="w-16 text-center border rounded px-1"
                  />
                  <div className="text-sm text-slate-600">subtotal: {formatCurrency(it.subtotal)}</div>
                </div>
              </div>

              <button
                onClick={() => onRemove(it.menu.id)}
                className="text-red-600 p-2 hover:bg-red-50 rounded"
                aria-label="eliminar"
                type="button"
              >
                <FaTrash />
              </button>
            </li>
          ))}
        </ul>
      )}

      <div className="mt-4">
        <div className="flex justify-between items-center font-bold">
          <div>Total</div>
          <div className="text-blue-700 text-lg">{formatCurrency(total)}</div>
        </div>

        <button
          onClick={onOpenConfirm}
          disabled={items.length === 0}
          className={`mt-3 w-full py-2 rounded-lg text-white ${items.length === 0 ? "bg-gray-300 cursor-not-allowed" : "bg-blue-700 hover:bg-blue-800"}`}
          type="button"
        >
          Confirmar orden
        </button>
      </div>
    </aside>
  );
};

export default OrderCart;

```
### 4.3 Crear componente ConfirmOrderModal.jsx

Modal para confirmar la orden: selecciona mesa y cliente (traídos del backend), muestra resumen, permite enviar orden. Devuelve la acción al padre (onConfirm(dto)).

```JavaScript
import React, { useEffect, useState } from "react";

const ConfirmOrderModal = ({ visible, onClose, items, total, onConfirm, token, fetchMesas, fetchClientes }) => {
  const [mesas, setMesas] = useState([]);
  const [clientes, setClientes] = useState([]);
  const [selectedMesa, setSelectedMesa] = useState(null);
  const [selectedCliente, setSelectedCliente] = useState(null);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (!visible) return;
    (async () => {
      try {
        const [mesasRes, clientesRes] = await Promise.all([fetchMesas(token), fetchClientes(token)]);
        setMesas(mesasRes || []);
        setClientes(clientesRes || []);
        if (mesasRes?.length) setSelectedMesa(mesasRes[0].id);
      } catch (e) {
        console.error(e);
      }
    })();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [visible]);

  const handleConfirm = async () => {
    if (!selectedMesa || !selectedCliente) {
      alert("Selecciona mesa y cliente antes de confirmar.");
      return;
    }
    const dto = {
      // DTO shape similar a backend: clienteDTO, mesaDTO, usuarioDTO se setearán en backend según token,
      // pero aquí mandamos lo mínimo requerido: cliente y mesa y detalleList
      clienteDTO: { id: selectedCliente },
      mesaDTO: { id: selectedMesa },
      total: total,
      detalleList: items.map((it) => ({
        menuDTO: { id: it.menu.id },
        cantidad: it.cantidad,
        precio: it.precio,
        subTotal: it.subtotal,
      })),
    };

    setLoading(true);
    try {
      await onConfirm(dto);
      onClose();
    } catch (err) {
      console.error(err);
      alert("Error al crear la orden. Revisa la consola.");
    } finally {
      setLoading(false);
    }
  };

  if (!visible) return null;

  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center p-4">
      <div className="absolute inset-0 bg-black/40" onClick={onClose} />
      <div className="relative z-10 w-full max-w-2xl bg-white rounded-xl shadow-lg p-6">
        <h3 className="text-xl font-semibold mb-3">Confirmar Orden</h3>

        <div className="grid grid-cols-1 md:grid-cols-2 gap-3 mb-4">
          <div>
            <label className="block text-sm text-slate-600 mb-1">Mesa</label>
            <select value={selectedMesa || ""} onChange={(e) => setSelectedMesa(Number(e.target.value))} className="w-full border rounded px-2 py-1">
              <option value="">-- Seleccione mesa --</option>
              {mesas.map((m) => (
                <option key={m.id} value={m.id}>{m.nombre || `Mesa ${m.numero || m.id}`}</option>
              ))}
            </select>
          </div>

          <div>
            <label className="block text-sm text-slate-600 mb-1">Cliente</label>
            <select value={selectedCliente || ""} onChange={(e) => setSelectedCliente(Number(e.target.value))} className="w-full border rounded px-2 py-1">
              <option value="">-- Seleccione cliente --</option>
              {clientes.map((c) => (
                <option key={c.id} value={c.id}>{c.nombre }</option>
              ))}
            </select>
          </div>
        </div>

        <div className="mb-4">
          <h4 className="font-medium">Resumen</h4>
          <div className="max-h-40 overflow-auto mt-2 divide-y">
            {items.map((it) => (
              <div key={it.menu.id} className="py-2 flex justify-between items-center">
                <div>
                  <div className="font-medium">{it.menu.nombre}</div>
                  <div className="text-sm text-slate-500">{it.cantidad} x ${Number(it.precio).toFixed(2)}</div>
                </div>
                <div className="font-semibold">${Number(it.subtotal).toFixed(2)}</div>
              </div>
            ))}
          </div>

          <div className="mt-3 text-right font-bold text-lg">Total: ${Number(total).toFixed(2)}</div>
        </div>

        <div className="flex gap-2 justify-end">
          <button onClick={onClose} className="px-4 py-2 rounded border">Cancelar</button>
          <button onClick={handleConfirm} disabled={loading} className="px-4 py-2 rounded bg-blue-700 text-white">
            {loading ? "Enviando..." : "Enviar a cocina"}
          </button>
        </div>
      </div>
    </div>
  );
};

export default ConfirmOrderModal;

```
### 4.4.- Crear MeseroView.jsx e intregar los componentes anteriores
Aquí integro se todo: trae menús reales, permite agregar al carrito, editar cantidades, abrir modal y crear orden con createOrden. 
### 4.5.- Crear componente OrdenesPage.jsx
### 4.6 Crear una en el archivo App.jsx
```JavaScript
{
   path: 'ordenes',
   element: <OrdenesPage />
}
```
### 4.7 Programar CocineroView.jsx
Ahora creamos el componente CocineroView.jsx, que es la vista del *cocinero* para gestionar las órdenes en preparación.

Este componente mostrará:

1. Un listado de órdenes en estado CREADA o PREPARANDO.
2. Detalles de cada orden (mesa, cliente, mesero, lista de productos).
3. Botones de acción para cambiar estado:
   * CREADA → PREPARANDO (cuando empieza a cocinar).
   * PREPARANDO → LISTA (cuando la orden está lista para entregar).

```JavaScript
import React, { useEffect, useState } from "react";
import axios from "axios";
import { useAuth } from "../context/AuthContext";
import { FaUtensils, FaCheckCircle } from "react-icons/fa";
import { urlBase } from "../../utils/config";

const CocineroView = () => {
  const { token } = useAuth();
  const [ordenes, setOrdenes] = useState([]);
  const [loading, setLoading] = useState(true);

  //  Obtenemos las órdenes pendientes
  const fetchOrdenes = async () => {
    try {
      setLoading(true);
      const response = await axios.get(`${urlBase}ordenes/estado/CREADA`, {
        headers: { Authorization: `Bearer ${token}` },
      });

      const resProceso = await axios.get(`${urlBase}ordenes/estado/PREPARANDO`, {
        headers: { Authorization: `Bearer ${token}` },
      });

      setOrdenes([...response.data, ...resProceso.data]);
    } catch (err) {
      console.error("Error cargando órdenes:", err);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchOrdenes();
  }, []);

  // Cambiar estado de la orden
  const cambiarEstado = async (orden, nuevoEstado) => {
    try {
      await axios.put(
        `${urlBase}ordenes/cambiar-estado`,
        { id: orden.id, estado: nuevoEstado },
        { headers: { Authorization: `Bearer ${token}` } }
      );
      fetchOrdenes(); // Actualizamos listado de ordenes
    } catch (err) {
      console.error("Error cambiando estado:", err);
    }
  };

  if (loading) {
    return <p className="text-center text-gray-500">Cargando órdenes...</p>;
  }

  return (
    <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
      {ordenes.length === 0 ? (
        <p className="text-gray-500">No hay órdenes pendientes</p>
      ) : (
        ordenes.map((orden) => (
          <div
            key={orden.id}
            className="bg-white shadow-lg rounded-2xl p-4 border border-gray-200"
          >
            {/* Encabezado */}
            <div className="flex justify-between items-center mb-3">
              <h2 className="text-lg font-bold text-blue-700">
                Orden #: {orden.correlativo}
              </h2>
              <span
                className={`px-3 py-1 text-sm rounded-full ${
                  orden.estado === "CREADA"
                    ? "bg-yellow-100 text-yellow-700"
                    : "bg-blue-100 text-blue-700"
                }`}
              >
                {orden.estado}
              </span>
            </div>

            {/* Información */}
            <p className="text-sm text-gray-600 mb-2">
              <strong>Mesa:</strong> {orden.mesaDTO?.numero}
            </p>
            <p className="text-sm text-gray-600 mb-2">
              <strong>Cliente:</strong> {orden.clienteDTO?.nombre}
            </p>
            <p className="text-sm text-gray-600 mb-4">
              <strong>Mesero:</strong> {orden.usuarioDTO?.nombre}
            </p>

            {/* Detalle */}
            <div className="bg-gray-50 rounded-lg p-3 mb-3">
              <h3 className="font-semibold text-gray-700 mb-2">Menus / Productos:</h3>
              <ul className="space-y-1">
                {orden.detalleList.map((detalle) => (
                  <li
                    key={detalle.id}
                    className="flex justify-between text-sm text-gray-700"
                  >
                    <span>
                      {detalle.cantidad} x {detalle.menuDTO?.nombre}
                    </span>
                    <span>${Number(detalle.cantidad * detalle.precio).toFixed(2)}</span>
                  </li>
                ))}
              </ul>
            </div>

            {/* Total */}
            <p className="text-right font-bold text-gray-800 mb-3">
              Total: ${Number(orden.total).toFixed(2)}
            </p>

            {/* Botones */}
            <div className="flex justify-end gap-2">
              {orden.estado === "CREADA" && (
                <button
                  onClick={() => cambiarEstado(orden, "PREPARANDO")}
                  className="flex items-center px-3 py-2 bg-yellow-500 text-white rounded-lg hover:bg-yellow-600"
                >
                  <FaUtensils className="mr-2" /> Iniciar
                </button>
              )}

              {orden.estado === "PREPARANDO" && (
                <button
                  onClick={() => cambiarEstado(orden, "LISTA")}
                  className="flex items-center px-3 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700"
                >
                  <FaCheckCircle className="mr-2" /> Marcar lista
                </button>
              )}
            </div>
          </div>
        ))
      )}
    </div>
  );
};  

export default CocineroView;

```

