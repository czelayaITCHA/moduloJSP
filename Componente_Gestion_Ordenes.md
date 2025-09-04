# Componente Gestión de Ordenes
## Requisitos del backend para crear componente de gestión de ordenes en el frontend
1. Agregar los roles: ADMIN, MESERO/A, COCINERO Y CAJERO
2. Insertar 4 usuarios con diferente rol, usar el endpoint api//auth/register
3. Haber implementado los endpoints para obtener listado de mesas y clientes

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
En la carpeta ordenes vamos a crear los archivos de los diferentes componentes para gestionar las ordenes, serán componentes separados que se integrarán después en un solo componente. Cuando termine la creación de los componentes su estructura debe verse como la siguiente imagen:

<img width="260" height="443" alt="image" src="https://github.com/user-attachments/assets/84cad721-989e-4688-b0cf-6ad190252226" />

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
import { useAuth } from "../context/AuthContext";
import Swal from "sweetalert2";

const ConfirmOrderModal = ({ visible, onClose, items, total, onConfirm, token, fetchMesas, fetchClientes }) => {

  const {user} = useAuth();
  
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
    
  }, [visible]);

  const handleConfirm = async () => {
    if (!selectedMesa || !selectedCliente) {
      Swal.fire("Advertencia!","Selecciona mesa y cliente antes de confirmar.","warning");
      return;
    }
    //construimos el json como lo espera el backend
    const dto = {
      total: total,
      clienteDTO: { id: selectedCliente },
      mesaDTO: { id: selectedMesa },
      usuarioDTO: {id: user?.userId},
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
      Swal.fire("Error","Error al crear la orden. Revisa la consola.", "error");
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
Aquí se integra todo: trae menús / productos, permite agregar a la orden, editar cantidades, abrir modal y crear orden con createOrden. 
```JavaScript
import React, { useEffect, useMemo, useState } from "react";
import { useAuth } from "../context/AuthContext";
import MenuCard from "./MenuCard";
import OrderCart from "./OrderCart";
import ConfirmOrderModal from "./ConfirmOrderModal";
import { getMenus, getMesas, getClientes, createOrden } from "../services/ordenService";
import Swal from "sweetalert2";

const MeseroView = () => {
  const { token } = useAuth();
  
  const [menus, setMenus] = useState([]);
  const [loadingMenus, setLoadingMenus] = useState(true);

  // orden en construcción: array { menu, cantidad, precio, subtotal }
  const [cart, setCart] = useState([]);
  const [isConfirmOpen, setIsConfirmOpen] = useState(false);
  const [isSubmitting, setIsSubmitting] = useState(false);

  useEffect(() => {
    const fetch = async () => {
      setLoadingMenus(true);
      try {
        const data = await getMenus(token);
        // aseguramos campos esperados
        setMenus(Array.isArray(data) ? data.filter(m=>m.disponible !== false) : []);
      } catch (e) {
        console.error("Error al obtener menus", e);
        setMenus([]);
      } finally {
        setLoadingMenus(false);
      }
    };
    if (token) fetch();
  }, [token]);

  const addToCart = (menu, cantidad) => {
    setCart((prev) => {
      const exists = prev.find((p) => p.menu.id === menu.id);
      if (exists) {
        return prev.map((p) =>
          p.menu.id === menu.id
            ? {
                ...p,
                cantidad: p.cantidad + cantidad,
                subtotal: Number(((p.cantidad + cantidad) * Number(menu.precioUnitario)).toFixed(2)),
              }
            : p
        );
      }
      return [
        ...prev,
        {
          menu,
          cantidad,
          precio: Number(menu.precioUnitario),
          subtotal: Number((cantidad * Number(menu.precioUnitario)).toFixed(2)),
        },
      ];
    });
  };

  const removeFromCart = (menuId) => {
    setCart((prev) => prev.filter((p) => p.menu.id !== menuId));
  };

  const updateQty = (menuId, qty) => {
    setCart((prev) =>
      prev.map((p) =>
        p.menu.id === menuId
          ? { ...p, cantidad: qty, subtotal: Number((qty * Number(p.precio)).toFixed(2)) }
          : p
      )
    );
  };

  const total = useMemo(() => cart.reduce((s, it) => s + Number(it.subtotal), 0), [cart]);

  const handleConfirm = async (dto) => {
    setIsSubmitting(true);
    try {
      // DTO que se envía al backend
      const response = await createOrden(dto, token);
      console.log("response", response)
      Swal.fire("Registrado", response?.message, "success");
      
      // limpiar carrito (Orden)
      setCart([]);
    } catch (err) {
      console.error(err);
      const msg = err?.response?.data?.message || "Error al crear la orden";
      Swal.fire("Error", msg, "error");
      throw err;
    } finally {
      setIsSubmitting(false);
    }
  };

  if (loadingMenus) {
    return <div className="p-6 text-center">Cargando menú...</div>;
  }

  return (
    <div className="p-4">
      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        {/* Menú: ocupa 2/3 en pantallas grandes */}
        <div className="lg:col-span-2">
          <header className="flex items-center justify-between mb-4">
            <h2 className="text-2xl font-bold text-slate-800">Menú</h2>
            <div className="text-slate-600">Toca para agregar al pedido</div>
          </header>

          <div className="grid grid-cols-2 sm:grid-cols-2 md:grid-cols-3 gap-4">
            {menus.map((m) => (
              <MenuCard key={m.id} menu={m} onAdd={addToCart} />
            ))}
          </div>
        </div>

        {/* Carrito o Orden */}
        <div>
          <OrderCart
            items={cart}
            onRemove={removeFromCart}
            onUpdateQty={updateQty}
            total={total}
            onOpenConfirm={() => setIsConfirmOpen(true)}
          />
        </div>
      </div>

      <ConfirmOrderModal
        visible={isConfirmOpen}
        onClose={() => setIsConfirmOpen(false)}
        items={cart}
        total={total}
        onConfirm={handleConfirm}
        token={token}
        fetchMesas={getMesas}
        fetchClientes={getClientes}
      />
    </div>
  );
};

export default MeseroView;

```
### 4.5.- Crear componente principal 
Ahora creamos el contenedor principal que se cargará cuando el usuario haga click en el menú *Gestión Ordenes*, a este componente le llamaremos *OrdenesPage.jsx*. Este componente será el *router inteligente* y hará lo siguiente:
1. Detecta el rol del usuario logueado desde useAuth()
2. Cargar el subcomponente correspondiente:
   * MeseroView.jsx → si el rol es MESERO/A
   * CocineroView.jsx → si el rol es COCINERO
   * CajeroView.jsx → si el rol es CAJERO
   * Si es ADMIN, muestra todos (o un dashboard de administración de órdenes).
```JavaScript
import React from "react";
import { useAuth } from "../context/AuthContext";
import MeseroView from "./MeseroView";
import CocineroView from "./CocineroView";
import CajeroView from "./CajeroView";

const OrdenesPage = () => {
  const { user } = useAuth();

  if (!user) {
    return <p className="text-center mt-10">Cargando usuario...</p>;
  }

  return (
    <div className="p-4">
      <h1 className="text-2xl font-bold text-blue-700 mb-6">
        Gestión de Órdenes
      </h1>

      {user.role === "MESERO/A" && <MeseroView />}
      {user.role === "COCINERO" && <CocineroView />}
      {user.role === "CAJERO" && <CajeroView />}
      {user.role === "ADMIN" && (
        <div className="space-y-10">
          <section>
            <h2 className="text-xl font-semibold text-gray-700 mb-4">
              Vista Mesero
            </h2>
            <MeseroView />
          </section>

          <section>
            <h2 className="text-xl font-semibold text-gray-700 mb-4">
              Vista Cocinero
            </h2>
            <CocineroView />
          </section>

          <section>
            <h2 className="text-xl font-semibold text-gray-700 mb-4">
              Vista Cajero
            </h2>
            <CajeroView />
          </section>
        </div>
      )}
    </div>
  );
};

export default OrdenesPage;
```
  
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
### 4.8 Programar componente CajeroView.jsx
Por el momento se dejará preparada la interfaz de usuario, porque aún no hay una entidad para registrar el pago en el Backend, pero posteriormente se hará. Este componente tendrá las funciones:

1. Mostrar card cuando la orden esté con estado LISTA, el cajero ve un botón "Registrar Pago".
2. Al presionar, se abre un modal elegante donde:
   * Se muestra el total de la orden.
   * Se elige el método de pago (select).
   * Se ingresa el monto pagado (por si el cliente paga más para dar cambio).
```JavaScript
import React, { useState, useEffect } from "react";
import axios from "axios";
import { Dialog } from "primereact/dialog";
import { Dropdown } from "primereact/dropdown";
import { Button } from "primereact/button";
import { InputNumber } from "primereact/inputnumber";
import { Card } from "primereact/card";
import { urlBase } from "../../utils/config";
import { useAuth } from "../context/AuthContext";

const CajeroView = () => {
  const {token}  = useAuth(); 
  const [ordenes, setOrdenes] = useState([]);
  const [selectedOrden, setSelectedOrden] = useState(null);
  const [showPago, setShowPago] = useState(false);
  const [montoRecibido, setMontoRecibido] = useState(null);
  const [metodoPago, setMetodoPago] = useState("EFECTIVO");
  const [ticketData, setTicketData] = useState(null);

  useEffect(() => {
    fetchOrdenes();
  }, []);

  const fetchOrdenes = async () => {
    try {
      const response = await axios.get(`${urlBase}ordenes/estado/LISTA`, {
        headers: { Authorization: `Bearer ${token}` },
      });
      setOrdenes(response.data);
    } catch (err) {
      console.error("Error cargando órdenes", err);
    }
  };

  const calcularCambio = () => {
    if (!montoRecibido || !selectedOrden) return 0;
    return (montoRecibido - selectedOrden.total).toFixed(2);
  };

  const registrarPago = async () => {
    try {
      const pago = {
        ordenId: selectedOrden.id,
        monto: selectedOrden.total,
        metodo: metodoPago,
        recibido: montoRecibido,
        cambio: calcularCambio(),
      };

      await axios.post("/api/pagos", pago); //pendiente de crear
      await axios.put(`/api/ordenes/${selectedOrden.id}/cambiar-estado`, {
        estado: "PAGADA",
      });

      setTicketData({ ...selectedOrden, pago });
      setShowPago(false);
      fetchOrdenes();
      abrirTicketEnNuevaVentana({ ...selectedOrden, pago });
    } catch (err) {
      console.error("Error registrando pago", err);
    }
  };

  const abrirTicketEnNuevaVentana = (orden) => {
    const ventana = window.open("", "Ticket", "width=400,height=600");
    ventana.document.write(`
      <html>
        <head>
          <title>Ticket</title>
          <style>
            body { font-family: monospace; padding: 20px; }
            h2 { text-align: center; }
            .total { font-weight: bold; margin-top: 10px; }
          </style>
        </head>
        <body>
          <h2>🧾 Restaurante</h2>
          <p><b>Orden #: </b> ${orden.correlativo}</p>
          <p><b>Cliente:</b> ${orden.clienteDTO.nombre || "N/A"}</p>
          <p><b>Mesa:</b> ${orden.mesa?.numero}</p>
          <hr/>
          ${orden.detalleOrden
            .map(
              (d) =>
                `<p>${d.cantidad} x ${d.menu.nombre} - $${d.subtotal.toFixed(
                  2
                )}</p>`
            )
            .join("")}
          <hr/>
          <p class="total">TOTAL: $${orden.total.toFixed(2)}</p>
          <p>Recibido: $${orden.pago.recibido}</p>
          <p>Cambio: $${orden.pago.cambio}</p>
          <p>Método: ${orden.pago.metodo}</p>
          <hr/>
          <p style="text-align:center;">¡Gracias por su compra!</p>
          <script>
            window.print();
          </script>
        </body>
      </html>
    `);
    ventana.document.close();
  };

  return (
    <div>
      <h2 className="text-xl font-semibold text-blue-700 mb-4">
        Cajero - Cobro de Órdenes
      </h2>

      <div className="grid md:grid-cols-2 lg:grid-cols-3 gap-4">
        {ordenes.map((orden) => (
          <Card
            key={orden.id}
            title={`Orden #${orden.correlativo}`}
            subTitle={`Cliente: ${orden.clienteDTO?.nombre || "N/A"}`}
            className="shadow-lg"
          >
            <p><b>Mesa:</b> {orden.mesaDTO?.numero}</p>
            <div className="mt-3">
                <p className="font-semibold">Detalle de la Orden:</p>
                <ul className="text-sm list-disc pl-5">
                {orden.detalleList?.map((d) => (
                    <li key={d.id}>
                    {d.cantidad} x {d.menuDTO?.nombre} - ${Number(d.cantidad * d.precio).toFixed(2)}
                    </li>
                ))}
                </ul>
            </div>
            <p><b className="font-semibold">Total:</b> <b className="font-extrabold text-3xl text-blue-600">${orden.total.toFixed(2)}</b></p>
            
            <Button
              label="Registrar Pago"
              icon="pi pi-dollar"
              className="p-button-success mt-3"
              onClick={() => {
                setSelectedOrden(orden);
                setShowPago(true);
              }}
            />
          </Card>
        ))}
      </div>

      <Dialog
        header="Registrar Pago"
        visible={showPago}
        style={{ width: "400px" }}
        modal
        onHide={() => setShowPago(false)}
      >
        {selectedOrden && (
          <div className="space-y-4">
            <p><b>Total a pagar:</b> ${selectedOrden.total.toFixed(2)}</p>
            <Dropdown
              value={metodoPago}
              options={["EFECTIVO", "TARJETA", "TRANSFERENCIA"]}
              onChange={(e) => setMetodoPago(e.value)}
              placeholder="Seleccione método"
              className="w-full"
            />
            <InputNumber
              value={montoRecibido}
              onValueChange={(e) => setMontoRecibido(e.value)}
              mode="currency"
              currency="USD"
              locale="es-SV"
              placeholder="Monto recibido"
              className="w-full"
            />
            <p><b>Cambio:</b> ${calcularCambio()}</p>
            <Button
              label="Confirmar Pago"
              icon="pi pi-check"
              className="p-button-success w-full"
              onClick={registrarPago}
              disabled={!montoRecibido || montoRecibido < selectedOrden.total}
            />
          </div>
        )}
      </Dialog>
    </div>
  );
};

export default CajeroView;

```

### Nota importante, asegurarse de tener los siguientes ENUM en el Backend para el estados de ordenes y métodos de pago

1. EstadoOrden
```Java
package com.devsoft.orders_api.utils;
public enum EstadoOrden {
    CREADA,
    CONFIRMADA,
    PREPARANDO,
    LISTA,
    ENTREGADA,
    PAGADA,
    ANULADA
}
```
3. MetodoPago
```Java
  package com.devsoft.orders_api.utils;

public enum MetodoPago {
    EFECTIVO,
    TARJETA,
    TRANSFERENCIA
}
``` 
