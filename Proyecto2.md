using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Diagnostics;
using System.Drawing;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace Enunciado_2
{
    public partial class Form1: Form
    {
        public Form1()
        {
            InitializeComponent();
            
        }

        //Creamos la clase Registro que nos servirá para almacenar los datos junto a la lista
        public class Registro
        {
            public DateTime g_fecha { get; set; }
            public string g_nombre { get; set; }
            public bool g_victoriOLose { get; set; } 
            public int g_puntosTotales { get; set; }

        }

        //Definimos la lista
        public List<Registro> g_listaDeVideojuegos = new List<Registro>();

        //Definimos el nombre indicado para el archivo
        public string g_archivo = ("base_de_informacion.txt");


        //Botón para añadir los datos a la lista
        public void btnAñadir_Click(object sender, EventArgs e)
        {
            //Asignar los valores de los datos a las variables
            string g_nombreGame = txtName.Text.Trim();
            bool g_victOlose = opVictoria.Checked;
            DateTime p_fecha = dtpFecha.Value;


            //Si no se completaron los campos, retorna
            if (string.IsNullOrEmpty(g_nombreGame) || p_fecha == null)
            {
                MessageBox.Show("No se completaron todos los campos");
                return;
            }

            //Calcula los puntos que se obtienen. Como g_victOLose es bool, si es true se le asignará el valor de
            //100, caso contrario será -20
            int g_puntosTot = g_victOlose ? 100 : -20;

            //Creamos el objeto registro para asignarle los datos obtenidos 
            Registro registro = new Registro
            {
                g_nombre = g_nombreGame,
                g_fecha = p_fecha,
                g_victoriOLose = g_victOlose,
                g_puntosTotales = g_puntosTot
            };

            //Añade el registro a la lista 
            g_listaDeVideojuegos.Add(registro);

            // Actualiza la lista de videojuegos en el formulario
            ActLista();
            
            //Limpiar datos
            LimpiarDatos();

            // Muestra el puntaje total acumulado
            PuntajeAcumulado();

            // Guarda los registros en el archivo
            RegistrarInArchivo();

        }

        //Mostrar el puntaje acumulado en un label
        public void PuntajeAcumulado()
        {
            // Calcula el puntaje total sumando los puntos de todos los registros
            int g_Acumulado = g_listaDeVideojuegos.Sum(registro => registro.g_puntosTotales);
            // Muestra el puntaje total en el formulario
            lblAcumulado.Text = g_Acumulado.ToString();
        }


        //Para actualizar la lista con los nuevos datos
        public void ActLista()
        {
            listGames.Items.Clear();

            foreach (Registro registro in g_listaDeVideojuegos)
            {
                //Creamos un elemento de ListView para añadirle los datos obtenidos y luego ir sobreescribiéndolos
                //en la lista junto con los datos anteriores usando el foreach
                ListViewItem linea = new ListViewItem(registro.g_nombre); 
                linea.SubItems.Add(registro.g_puntosTotales.ToString());
                linea.SubItems.Add(registro.g_victoriOLose ? "Victoria" : "Derrota");
                linea.SubItems.Add(registro.g_fecha.ToShortDateString());
                listGames.Items.Add(linea);
            }
        }

        //Limpia el formulario
        public void LimpiarDatos()
        {
            dtpFecha.Value = DateTime.Now;
            txtName.Text = string.Empty;
            opVictoria.Checked = true;
            opDerrota.Checked = false;

        }

        //Registrar los datos en el archivo indicado
        public void RegistrarInArchivo()
        {
            try
            {
                using (StreamWriter writer = new StreamWriter(g_archivo))
                {
                    foreach (Registro registro in g_listaDeVideojuegos)
                    {
                        //Escribimos la linea de la lista en el archivo
                        string linea = $"{registro.g_nombre}, {registro.g_puntosTotales}, {registro.g_victoriOLose}, {registro.g_fecha}";
                        writer.WriteLine(linea);
                    }
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show("Error, no se pudo guardar" + ex.Message);
            }
        }

        //Botón para abrir el archivo de texto creado
        private void btnAbrirArchivo_Click(object sender, EventArgs e)
        {
            //Creamos el proceso abrir archivo
            Process abrirarchivo = new Process();
            //Obtiene la información, el nombre del documento y lo abre
            abrirarchivo.StartInfo.FileName = "base_de_informacion.txt";
            //abre el documento
            abrirarchivo.Start();
        }

        //Botón para eliminar un elemento de la lista
        private void btnQuitarLista_Click(object sender, EventArgs e)
        {
            //Si no se selecciona ningún elemento, muestra error
            if (listGames.SelectedItems.Count > 0)
            {
                foreach (ListViewItem linea in listGames.SelectedItems)
                {
                    // Obtiene los datos del registro seleccionado
                    string p_nombreJuego = linea.SubItems[0].Text;
                    int p_puntosObtenidos = int.Parse(linea.SubItems[1].Text);
                    bool p_victoria = linea.SubItems[2].Text == "Victoria";
                    DateTime p_fecha = DateTime.Parse(linea.SubItems[3].Text);

                    // Busca los registros que coinciden con los datos obtenidos
                    List<Registro> registrosAEliminar = g_listaDeVideojuegos.Where(registro =>
                        registro.g_nombre == p_nombreJuego &&
                        registro.g_fecha.Date.Equals(p_fecha.Date) &&
                        registro.g_victoriOLose == p_victoria &&
                        registro.g_puntosTotales == p_puntosObtenidos).ToList();

                    foreach (Registro registro in registrosAEliminar)
                    {
                        g_listaDeVideojuegos.Remove(registro);
                    }
                    // Remueve el elemento de la lista en el formulario
                    listGames.Items.Remove(linea);
                }

                // Muestra el puntaje total actualizado
                PuntajeAcumulado();

                // Guarda los registros en el archivo
                RegistrarInArchivo();
            }
            else
            {
                MessageBox.Show("Selecciona al menos un videojuego para eliminar.", "Error", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }

        }

        //botón para cerrar el programa
        private void btnSalir_Click(object sender, EventArgs e)
        {
            //cerramos la aplicación
            Application.Exit();
        }

        private void Form1_Load(object sender, EventArgs e)
        {

        }
    }
}
