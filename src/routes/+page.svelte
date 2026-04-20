<script>
  import { supabase } from '$lib/supabaseClient';

  let fechaInicio = $state('');
  let fechaFin = $state('');
  
  /** @type {any[]} */ 
  let comisiones = $state([]); // Aquí le decimos que es un arreglo de cualquier cosa
  let cargando = $state(false);

  async function calcular() {
    cargando = true;
    
    const { data: vendedores } = await supabase.from('Vendedor').select('*');
    const { data: reglas } = await supabase.from('Reglas').select('*');
    const { data: ventas } = await supabase
      .from('Ventas')
      .select('*')
      .gte('fecha_venta', fechaInicio)
      .lte('fecha_venta', fechaFin);

    if (vendedores && reglas && ventas) {
      comisiones = vendedores.map(v => {
        const misVentas = ventas.filter(venta => venta.vendedor_id === v.id);
        let totalComision = 0;

        misVentas.forEach(venta => {
          /** @type {any} */
          let reglaGanadora = null; // Le avisamos que aquí guardaremos una regla
          
          reglas.forEach(r => {
            if (venta.monto >= r.amount) {
              reglaGanadora = r;
            }
          });

          // Ahora TypeScript ya no dirá que es 'never' porque sabe que puede ser una regla
          if (reglaGanadora) {
            totalComision += venta.monto * reglaGanadora.rule;
          }
        });

        return { nombre: v.nombre, total: totalComision };
      });
    }

    cargando = false;
  }
</script>

<main>
  <h1>Calculadora UDLA - Comisiones</h1>

  <section class="controles">
    <input type="date" bind:value={fechaInicio} />
    <input type="date" bind:value={fechaFin} />
    
    <button onclick={calcular} disabled={cargando}>
      {cargando ? 'Procesando...' : 'Calcular'}
    </button>
  </section>

  <section class="resultados">
    {#each comisiones as c}
      <div class="card">
        <strong>{c.nombre}:</strong> ${c.total.toFixed(2)}
      </div>
    {:else}
      <p>Selecciona fechas y presiona calcular.</p>
    {/each}
  </section>
</main>

<style>
  main { max-width: 400px; margin: 40px auto; font-family: sans-serif; text-align: center; }
  .controles { display: flex; flex-direction: column; gap: 10px; margin-bottom: 20px; }
  .card { background: #eee; padding: 15px; margin: 10px 0; border-radius: 8px; }
  button { background: #ff3e00; color: white; border: none; padding: 10px; cursor: pointer; }
  button:disabled { background: #ccc; }
</style>