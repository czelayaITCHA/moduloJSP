# Reporte de Ingesos usando libreria itextpdf
Indrodución

## 1. Instalación de dependencia itextpdf
Editar el archivo pom.xml y agregar dependencia, como se muestra a continucaión 
```xml
<dependency>
      <groupId>com.itextpdf</groupId>
			<artifactId>itextpdf</artifactId>
			<version>5.5.13.4</version>
</dependency>
```
## 2. Crear DTO para el resultado de la consulta

```Java
package com.devsoft.orders_api.dto;

import lombok.AllArgsConstructor;
import lombok.Getter;
import java.math.BigDecimal;
import java.time.LocalDate;

@Getter
@AllArgsConstructor
public class IngresoDTO {
    private Long ordenId;
    private String correlativo;
    private LocalDate fechaPago;
    private BigDecimal totalOrden;
    private String clienteNombre;
    private Integer mesaNumero;
    private String meseroNombre;
    private BigDecimal sumaPagos;
}

```
## 3. Crear método con query para obtener los resultados del reporte en el respositorio PagoRepository
```Java
@Query("SELECT new com.devsoft.orders_api.dto.IngresoDTO(" +
            "o.id, o.correlativo, p.fechaPago, o.total, c.nombre, m.numero, u.nombre, SUM(p.monto)) " +
            "FROM Pago p " +
            "JOIN p.orden o " +
            "JOIN o.cliente c " +
            "JOIN o.mesa m " +
            "JOIN o.usuario u " +
            "WHERE p.fechaPago BETWEEN :fechaInicio AND :fechaFin " +
            "GROUP BY o.id, o.correlativo, p.fechaPago, o.total, c.nombre, m.numero, u.nombre " +
            "ORDER BY p.fechaPago ASC")
    List<IngresoDTO> obtenerIngresosRangoFechas(
            @Param("fechaInicio") LocalDate fechaInicio,
            @Param("fechaFin") LocalDate fechaFin);
```
## 4. Programar service que arma los datos y genera PDF (iText)
Aquí es donde se construye el reporte con los datos que se obtienen del repositorio, usando clases y elementos de iTextPdf

```Java
@Service
@RequiredArgsConstructor
public class ReporteService {
    @Autowired
    private PagoRepository pagoRepository;

    //método que genera el pdf y devuelve los bytes
    public byte[] generarReporte(LocalDate inicio, LocalDate fin, boolean detallado) throws Exception {
        //obtenemos los datos para el reporte
        List<IngresoDTO> data = pagoRepository.obtenerIngresosRangoFechas(inicio, fin);

        Document document = new Document(PageSize.LETTER, 36, 36, 72, 36); // márgenes
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        PdfWriter writer = PdfWriter.getInstance(document, baos);

        // Evento para numeración de páginas
        writer.setPageEvent(new PdfPageEventHelper() {
            @Override
            public void onEndPage(PdfWriter writer, Document document) {
                ColumnText.showTextAligned(
                        writer.getDirectContent(),
                        Element.ALIGN_CENTER,
                        new Phrase(String.format("Página %d", writer.getPageNumber()),
                                FontFactory.getFont(FontFactory.HELVETICA, 9, Font.ITALIC)),
                        (document.right() - document.left()) / 2 + document.leftMargin(),
                        document.bottom() - 20,
                        0
                );
            }
        });

        document.open();

        // ========= Encabezado =========
        PdfPTable header = new PdfPTable(2);
        header.setWidthPercentage(100);
        header.setWidths(new int[]{1, 4});

        try {
            Image logo = Image.getInstance("src/main/resources/static/logo.jpg");
            logo.scaleToFit(70, 70);
            PdfPCell logoCell = new PdfPCell(logo, false);
            logoCell.setBorder(Rectangle.NO_BORDER);
            logoCell.setHorizontalAlignment(Element.ALIGN_LEFT);
            header.addCell(logoCell);
        } catch (Exception e) {
            PdfPCell emptyCell = new PdfPCell(new Phrase(" "));
            emptyCell.setBorder(Rectangle.NO_BORDER);
            header.addCell(emptyCell);
        }

        PdfPCell datosEmpresa = new PdfPCell();
        datosEmpresa.setBorder(Rectangle.NO_BORDER);
        datosEmpresa.addElement(new Paragraph("Mi Restaurante S.A. de C.V",
                FontFactory.getFont(FontFactory.HELVETICA_BOLD, 14)));
        datosEmpresa.addElement(new Paragraph("Dirección: Bario El Centro, Chalatenango",
                FontFactory.getFont(FontFactory.HELVETICA, 10)));
        datosEmpresa.addElement(new Paragraph("Tel: 2301-3333",
                FontFactory.getFont(FontFactory.HELVETICA, 10)));
        header.addCell(datosEmpresa);

        document.add(header);
        document.add(new Paragraph(" "));

        Paragraph titulo = new Paragraph("Reporte de Ingresos",
                FontFactory.getFont(FontFactory.HELVETICA_BOLD, 16));
        titulo.setAlignment(Element.ALIGN_CENTER);
        document.add(titulo);

        Paragraph rango = new Paragraph("Del " + inicio + " al " + fin,
                FontFactory.getFont(FontFactory.HELVETICA, 11));
        rango.setAlignment(Element.ALIGN_CENTER);
        document.add(rango);
        document.add(new Paragraph(" "));

        // ========= Contenido =========
        if (detallado) {
            PdfPTable table = new PdfPTable(7);
            table.setWidthPercentage(100);
            table.setWidths(new float[]{1.2f, 1.5f, 1.5f, 2f, 1.2f, 2f, 1.5f});

            String[] headers = {"No", "Correlativo", "Fecha Pago", "Cliente", "Mesa", "Mesero", "Monto Pagado"};
            for (String h : headers) {
                PdfPCell cell = new PdfPCell(new Phrase(h, FontFactory.getFont(FontFactory.HELVETICA_BOLD, 10)));
                cell.setBackgroundColor(BaseColor.LIGHT_GRAY);
                cell.setHorizontalAlignment(Element.ALIGN_CENTER);
                table.addCell(cell);
            }

            for (IngresoDTO dto : data) {
                table.addCell(dto.getOrdenId().toString());
                table.addCell(dto.getCorrelativo());
                table.addCell(dto.getFechaPago().toString());
                table.addCell(dto.getClienteNombre());
                table.addCell(dto.getMesaNumero().toString());
                table.addCell(dto.getMeseroNombre());
                table.addCell("$ " + dto.getSumaPagos().setScale(2, RoundingMode.HALF_UP).toString());
            }

            document.add(table);

        } else {
            // resumen agrupado por fechaPago
            Map<LocalDate, BigDecimal> resumen = data.stream()
                    .collect(Collectors.groupingBy(
                            IngresoDTO::getFechaPago,
                            Collectors.reducing(BigDecimal.ZERO, IngresoDTO::getSumaPagos, BigDecimal::add)
                    ));

            PdfPTable table = new PdfPTable(2);
            table.setWidthPercentage(50);
            table.setHorizontalAlignment(Element.ALIGN_CENTER);
            table.setWidths(new float[]{2f, 2f});

            String[] headers = {"Fecha", "Total Día"};
            for (String h : headers) {
                PdfPCell cell = new PdfPCell(new Phrase(h, FontFactory.getFont(FontFactory.HELVETICA_BOLD, 11)));
                cell.setBackgroundColor(BaseColor.LIGHT_GRAY);
                cell.setHorizontalAlignment(Element.ALIGN_CENTER);
                table.addCell(cell);
            }

            for (Map.Entry<LocalDate, BigDecimal> entry : resumen.entrySet()) {
                table.addCell(entry.getKey().toString());
                table.addCell("$ " + entry.getValue().setScale(2, RoundingMode.HALF_UP).toString());
            }

            document.add(table);
        }

        // ========= Total general =========
        BigDecimal totalGeneral = data.stream()
                .map(IngresoDTO::getSumaPagos)
                .reduce(BigDecimal.ZERO, BigDecimal::add);

        document.add(new Paragraph(" "));
        Paragraph total = new Paragraph("TOTAL GENERAL: $ " + totalGeneral.setScale(2, RoundingMode.HALF_UP),
                FontFactory.getFont(FontFactory.HELVETICA_BOLD, 12, BaseColor.BLACK));
        total.setAlignment(Element.ALIGN_RIGHT);
        document.add(total);

        document.close();
        return baos.toByteArray();
    }
}

```
## 5. Programar el endpoint en el controlador

```Java
@RestController
@RequestMapping("/api/reportes")
@RequiredArgsConstructor
public class ReporteController {
    @Autowired
    private ReporteService reporteService;

    @GetMapping("/ingresos")
    public ResponseEntity<byte[]> generarReporte(
            @RequestParam("fechaInicio") @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate fechaInicio,
            @RequestParam("fechaFinal") @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate fechaFinal,
            @RequestParam(defaultValue = "true") boolean detallado) throws Exception {

        byte[] pdf = reporteService.generarReporte(fechaInicio, fechaFinal, detallado);

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_PDF);
        headers.setContentDispositionFormData("filename", "reporte_ingresos.pdf");

        return new ResponseEntity<>(pdf, headers, HttpStatus.OK);
    }
}
```

## 6. Hacer pruebas es postman
Antes de crear un componente en react, asegurarse de probar en postman el resultado del endpoint reado en el punto anterior, usa método GET con esta url 

http://localhost:8080/api/reportes/ingresos?fechaInicio=2025-09-01&fechaFinal=2025-09-08&detallado=true

## 6. Crear Componente interfaz de usuario en el proyecto Frontend
* crear un Wrapper para ....
```JavaScript
import React, { useState } from "react";

const ReporteWrapper = ({ children, onGenerar, mostrarBoton = true }) => {
  const [loading, setLoading] = useState(false);
  const [pdfUrl, setPdfUrl] = useState(null);

  const handleGenerar = async () => {
    setLoading(true);
    try {
      const url = await onGenerar(); // Retorna un blob o una URL
      setPdfUrl(url);
    } catch (e) {
      console.error("Error generando reporte", e);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="p-6 bg-white shadow rounded space-y-4">
      <div>{children}</div>
      {mostrarBoton && (
        <button
          onClick={handleGenerar}
          disabled={loading}
          className="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700"
        >
          {loading ? "Generando..." : "Generar Reporte"}
        </button>
      )}
      {pdfUrl && (
        <iframe
          src={pdfUrl}
          title="Vista previa PDF"
          className="w-full h-[600px] border mt-4"
        />
      )}
    </div>
  );
};

export default ReporteWrapper;
```
* Crear componente ReporteIngresos
```JavaScript
import React, { useState } from "react";
import axios from "axios";
import ReporteWrapper from "./ReporteWrapper";
import { Calendar } from "primereact/calendar";
import { Button } from "primereact/button";
import { Card } from "primereact/card";
import { Dialog } from "primereact/dialog";
import { ProgressSpinner } from "primereact/progressspinner";
import { RadioButton } from "primereact/radiobutton";
import { useNavigate } from "react-router-dom";
import { urlBase } from "../../utils/config";
import Swal from "sweetalert2";

const ReporteIngresos = () => {
  const token = localStorage.getItem("token");
  const navigate = useNavigate();
  const [fechaInicio, setFechaInicio] = useState(null);
  const [fechaFinal, setFechaFinal] = useState(null);
  const [detallado, setDetallado] = useState(true);
  const [loading, setLoading] = useState(false);

  const verReportePdf = async () => {
    if (!fechaInicio || !fechaFinal) {
      Swal.fire("Precaución", "Por favor, seleccione el rango de fechas.", "warning");
      return;
    }
    if (fechaFinal < fechaInicio) {
      Swal.fire("Precaución", "La fecha final no puede ser anterior a la fecha inicial.", "warning");
      return;
    }

    const inicio = fechaInicio.toISOString().split("T")[0]; // yyyy-MM-dd
    const fin = fechaFinal.toISOString().split("T")[0];     // yyyy-MM-dd

    const endpoint = `${urlBase}reportes/ingresos?fechaInicio=${inicio}&fechaFinal=${fin}&detallado=${detallado}`;

    try {
      setLoading(true);
      const response = await axios.get(endpoint, {
        headers: { Authorization: `Bearer ${token}` },
        responseType: "blob",
      });

      const blob = new Blob([response.data], { type: "application/pdf" });
      const url = window.URL.createObjectURL(blob);

      // se abre en una nueva pestaña
      window.open(url, "_blank");

      window.URL.revokeObjectURL(url);
    } catch (error) {
      console.error("Error al generar el PDF:", error);
      Swal.fire("Error al generar el reporte.");
    } finally {
      setLoading(false);
    }
  };

  return (
    <ReporteWrapper mostrarBoton={false}>
      <div className="mt-0">
        <Button
          label="Regresar"
          icon="pi pi-arrow-left"
          severity="secondary"
          className="w-full md:w-auto"
          onClick={() => navigate("/reportes")}
        />
      </div>

      <div className="p-6 max-w-5xl mx-auto">
        <Card title="Reporte de Ingresos" className="shadow-xl rounded-2xl">
          {/* Filtros */}
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4 p-4">
            <div>
              <label className="font-semibold text-gray-700">Fecha Inicio</label>
              <Calendar
                value={fechaInicio}
                onChange={(e) => setFechaInicio(e.value)}
                showIcon
                dateFormat="yy-mm-dd"
                className="w-full"
                placeholder="Seleccione fecha"
              />
            </div>
            <div>
              <label className="font-semibold text-gray-700">Fecha Final</label>
              <Calendar
                value={fechaFinal}
                onChange={(e) => setFechaFinal(e.value)}
                showIcon
                dateFormat="yy-mm-dd"
                className="w-full"
                placeholder="Seleccione fecha"
              />
            </div>
          </div>

          {/* Selector de tipo reporte */}
          <div className="flex gap-6 p-4">
            <div className="flex items-center">
              <RadioButton
                inputId="detallado"
                name="tipo"
                value={true}
                onChange={(e) => setDetallado(e.value)}
                checked={detallado === true}
              />
              <label htmlFor="detallado" className="ml-2">Detallado</label>
            </div>
            <div className="flex items-center">
              <RadioButton
                inputId="resumido"
                name="tipo"
                value={false}
                onChange={(e) => setDetallado(e.value)}
                checked={detallado === false}
              />
              <label htmlFor="resumido" className="ml-2">Resumido</label>
            </div>
          </div>

          {/* Botón generar */}
          <div className="flex flex-wrap gap-3 justify-end p-4">
            <Button
              icon="pi pi-file-pdf"
              severity="danger"
              label="Generar Reporte"
              onClick={verReportePdf}
            />
          </div>
        </Card>

        {/* Dialog cargando */}
        <Dialog visible={loading} closable={false} modal header="Espere un momento">
          <div className="flex align-items-center justify-content-center">
            <ProgressSpinner />
            <span className="ml-3 text-blue-700">Generando Reporte...</span>
          </div>
        </Dialog>
      </div>
    </ReporteWrapper>
  );
};

export default ReporteIngresos;

```
* Crear ruta
 
```JavaScript
 {
	path: 'reporte-ingresos',
    element: <ReporteIngresos />
}
```
  
