# Gerenciador-Tarefas
Gerenciador-Tarefas
1. Model (Tarefa.cs)
using System;

namespace GerenciadorTarefas.Models
{
    public class Tarefa
    {
        public int Id { get; set; }
        public string Titulo { get; set; }
        public string Descricao { get; set; }
        public DateTime Data { get; set; }
        public string Status { get; set; } // Pendente, Concluída, etc.
    }
}

2. DbContext (AppDbContext.cs)
using Microsoft.EntityFrameworkCore;
using GerenciadorTarefas.Models;

namespace GerenciadorTarefas.Data
{
    public class AppDbContext : DbContext
    {
        public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

        public DbSet<Tarefa> Tarefas { get; set; }
    }
}


No Program.cs (ou Startup.cs se for versão antiga), registre o contexto:

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();


E adicione a connection string no appsettings.json:

"ConnectionStrings": {
  "DefaultConnection": "Server=localhost;Database=GerenciadorTarefasDB;Trusted_Connection=True;"
}

3. Controller (TarefaController.cs)
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using GerenciadorTarefas.Data;
using GerenciadorTarefas.Models;

namespace GerenciadorTarefas.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class TarefaController : ControllerBase
    {
        private readonly AppDbContext _context;

        public TarefaController(AppDbContext context)
        {
            _context = context;
        }

        // GET /Tarefa/{id}
        [HttpGet("{id}")]
        public async Task<IActionResult> ObterPorId(int id)
        {
            var tarefa = await _context.Tarefas.FindAsync(id);
            if (tarefa == null) return NotFound();
            return Ok(tarefa);
        }

        // GET /Tarefa/ObterTodos
        [HttpGet("ObterTodos")]
        public async Task<IActionResult> ObterTodos()
        {
            var tarefas = await _context.Tarefas.ToListAsync();
            return Ok(tarefas);
        }

        // GET /Tarefa/ObterPorTitulo?titulo=...
        [HttpGet("ObterPorTitulo")]
        public async Task<IActionResult> ObterPorTitulo([FromQuery] string titulo)
        {
            var tarefas = await _context.Tarefas
                .Where(t => t.Titulo.Contains(titulo))
                .ToListAsync();
            return Ok(tarefas);
        }

        // GET /Tarefa/ObterPorData?data=yyyy-MM-dd
        [HttpGet("ObterPorData")]
        public async Task<IActionResult> ObterPorData([FromQuery] DateTime data)
        {
            var tarefas = await _context.Tarefas
                .Where(t => t.Data.Date == data.Date)
                .ToListAsync();
            return Ok(tarefas);
        }

        // GET /Tarefa/ObterPorStatus?status=...
        [HttpGet("ObterPorStatus")]
        public async Task<IActionResult> ObterPorStatus([FromQuery] string status)
        {
            var tarefas = await _context.Tarefas
                .Where(t => t.Status.Equals(status, StringComparison.OrdinalIgnoreCase))
                .ToListAsync();
            return Ok(tarefas);
        }

        // POST /Tarefa
        [HttpPost]
        public async Task<IActionResult> Criar([FromBody] Tarefa tarefa)
        {
            _context.Tarefas.Add(tarefa);
            await _context.SaveChangesAsync();
            return CreatedAtAction(nameof(ObterPorId), new { id = tarefa.Id }, tarefa);
        }

        // PUT /Tarefa/{id}
        [HttpPut("{id}")]
        public async Task<IActionResult> Atualizar(int id, [FromBody] Tarefa tarefaAtualizada)
        {
            var tarefa = await _context.Tarefas.FindAsync(id);
            if (tarefa == null) return NotFound();

            tarefa.Titulo = tarefaAtualizada.Titulo;
            tarefa.Descricao = tarefaAtualizada.Descricao;
            tarefa.Data = tarefaAtualizada.Data;
            tarefa.Status = tarefaAtualizada.Status;

            await _context.SaveChangesAsync();
            return NoContent();
        }

        // DELETE /Tarefa/{id}
        [HttpDelete("{id}")]
        public async Task<IActionResult> Deletar(int id)
        {
            var tarefa = await _context.Tarefas.FindAsync(id);
            if (tarefa == null) return NotFound();

            _context.Tarefas.Remove(tarefa);
            await _context.SaveChangesAsync();
            return NoContent();
        }
    }
}

4. Migration

Após implementar, abra o Package Manager Console e rode:

Add-Migration Inicial
Update-Database


Isso vai criar a tabela Tarefas no seu banco de dados.

5. Testando com Swagger

No Program.cs, certifique-se de habilitar Swagger:

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();
