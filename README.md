# Ejercicios-Consulta-BD
GUIA 7

Ejericio 1

CREATE TABLE medicos_servicio_comunidad
AS SELECT u.nombre "UNIDAD",
                       m.pnombre||' '||m.apaterno||' '||m.amaterno "MEDICO",
                       UPPER(SUBSTR(u.nombre,1,2))||SUBSTR(m.apaterno,- 3,2)||SUBSTR(m.telefono,1,3)||TO_CHAR(m.fecha_contrato,'yyyy')||'@medicocktk.cl' "CORREO MEDICO",
                       COUNT(a.med_rut) "ATENCIONES MEDICAS"
FROM unidad u JOIN medico m
ON(u.uni_id=m.uni_id)
LEFT OUTER JOIN atencion a
ON(m.med_rut=a.med_rut)
AND TO_CHAR(fecha_atencion,'yyyy')=TO_CHAR(SYSDATE,'yyyy')-1
GROUP BY u.nombre, m.pnombre,m.apaterno,m.amaterno,m.telefono,m.fecha_contrato
HAVING COUNT(a.med_rut) < (SELECT MAX(COUNT(med_rut))
                                                          FROM atencion
                                                          WHERE TO_CHAR(fecha_atencion,'yyyy')=TO_CHAR(SYSDATE,'yyyy')-1
                                                          GROUP BY med_rut)
ORDER BY "UNIDAD",m.apaterno,m.amaterno;

Ejercicio 2

2.1

SELECT TO_CHAR(a.fecha_atencion,'mm/yyyy') "MES Y AÑO",
                 COUNT(a.ate_id) "TOTAL ATENCIONES",
                 TO_CHAR(SUM(a.costo),'$999G999G999') "VALOR TOTAL DE LAS ATENCIONES"
FROM atencion a
WHERE TO_CHAR(fecha_atencion,'yyyy')=TO_CHAR(SYSDATE,'yyyy')-1
GROUP BY TO_CHAR(fecha_atencion,'mm/yyyy')
HAVING COUNT(a.ate_id) > (SELECT ROUND(AVG(COUNT(ate_id)))
                                                     FROM atencion
                                                     WHERE TO_CHAR(fecha_atencion,'yyyy')=TO_CHAR(SYSDATE,'yyyy')-1
                                                     GROUP BY TO_CHAR(fecha_atencion,'mm/yyyy'))
ORDER BY TO_CHAR(fecha_atencion,'mm/yyyy');

2.2

SELECT p.pac_rut||'-'||p.dv_rut "RUT PACIENTE",
                p.pnombre||' '||p.snombre||' '||p.apaterno||' '||p.amaterno "NOMBRE PACIENTE",
                a.ate_id "ID ATENCION",
                pa.fecha_venc_pago "FECHA VENCIMIENTO PAGO",
                pa.fecha_pago "FECHA PAGO",
                pa.fecha_pago-pa.fecha_venc_pago "DIAS MOROSIDAD",
                TO_CHAR((pa.fecha_pago-pa.fecha_venc_pago)*2000,'$9G999G999')"VALOR MULTA"
FROM paciente p JOIN atencion a
ON(p.pac_rut=a.pac_rut)
JOIN pago_atencion pa
ON(pa.ate_id=a.ate_id)
WHERE TO_CHAR(pa.fecha_venc_pago,'YYYY')=TO_CHAR(SYSDATE,'YYYY')-1
GROUP BY p.pac_rut,p.dv_rut,p.pnombre,p.snombre,p.apaterno,p.amaterno,a.ate_id,pa.fecha_venc_pago,pa.fecha_pago
HAVING pa.fecha_pago-pa.fecha_venc_pago > (SELECT ROUND(AVG(fecha_pago-fecha_venc_pago))
                                                                                  FROM pago_atencion
                                                                                  WHERE TO_CHAR(fecha_venc_pago,'YYYY')=TO_CHAR(SYSDATE,'YYYY')-1
                                                                                  AND fecha_pago-fecha_venc_pago > 0)
ORDER BY "FECHA VENCIMIENTO PAGO","DIAS MOROSIDAD" DESC;

Ejercicio 3

CREATE TABLE resumen_atenciones_entsalud
AS SELECT ts.descripcion||','||s.descripcion "SISTEMA_SALUD",
                COUNT(p.pac_rut) "TOTAL ATENCIONES"
FROM tipo_salud ts JOIN salud s
ON(ts.tipo_sal_id=s.tipo_sal_id)
JOIN paciente p
ON(p.sal_id=s.sal_id)
JOIN atencion a
ON(a.pac_rut=p.pac_rut)
WHERE TO_CHAR(fecha_atencion,'mm/yyyy') = TO_CHAR(ADD_MONTHS(SYSDATE,-3),'mm/yyyy')
GROUP BY ts.descripcion,s.descripcion
HAVING COUNT(p.pac_rut) > (SELECT ROUND(AVG(COUNT(pac_rut)))
                                                        FROM atencion
                                                        WHERE TO_CHAR(fecha_atencion,'mm/yyyy') = TO_CHAR(ADD_MONTHS(SYSDATE,-3),'mm/yyyy')
                                                        GROUP BY EXTRACT(DAY FROM fecha_atencion))
ORDER BY "SISTEMA_SALUD";
