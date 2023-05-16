#Alexandre Faister 
#Cesar Albuquerque 
#Paula Henriques

import random
import os
import pandas as pd
from tabulate import tabulate

memory_physical = {}
for i in range(0, 1024 * 1024, 4096):
  memory_physical[i] = None

memory_virtual = {}
for i in range(0, 100 * 1024, 4096):
  memory_virtual[i] = None

swap_file = "swap.bin"
swap_file_size = 2**20  
with open(swap_file, "wb") as f:
  f.seek(swap_file_size - 1)
  f.write(b'\0')

def write_block_to_swap(block):
  with open(swap_file, "rb+") as f:
    f.seek(0, os.SEEK_END)  
    end_position = f.tell()
    f.write(block)  
  return end_position

def read_block_from_swap(position, size):
  with open(swap_file, "rb") as f:
    f.seek(position)  
    block = f.read(size)  
  return block

blocks = []
for i in range(20):
  size = random.randint(1, 10) * 4096  
  blocks.append(chr(random.randint(97, 122)) * size)

for i, block in enumerate(blocks):
  memory_virtual[i * 4096] = block

for address, block in memory_virtual.items():
  if block is not None and memory_physical.get(address) is None:
    block_size = len(block)
    found_empty_block = False
    for start_address in range(0, 1024 * 1024, 4096):
      if memory_physical.get(start_address) is None:
        end_address = start_address + block_size - 1
        if end_address < 1024 * 1024 and all(
            memory_physical.get(a) is None
            for a in range(start_address, end_address + 1)):
          found_empty_block = True
          break
    if found_empty_block:
      for a in range(start_address, end_address + 1):
        memory_physical[a] = block
      memory_virtual[address] = None
    else:
      position = write_block_to_swap(block.encode())
      memory_virtual[address] = {"position": position, "size": block_size}

for i in range(50):
  address = random.choice(list(memory_virtual.keys()))
  block = memory_virtual[address]
memory_physical_data = []
for address, block in memory_physical.items():
  if block is None:
    memory_physical_data.append([address, "Livre", "-"])
  else:
    memory_physical_data.append([address, "Alocado", block])

memory_virtual_data = []
for address, block in memory_virtual.items():
  if block is None:
    memory_virtual_data.append([address, "Livre", "-"])
  else:
    if isinstance(block, str):
      memory_virtual_data.append([address, "Alocado (Física)", block])
    else:
      memory_virtual_data.append([
        address, "Alocado (Swap)",
        f"Posição: {block['position']}, Tamanho: {block['size']}"
      ])

df_physical = pd.DataFrame(memory_physical_data,
                           columns=["Endereço", "Estado", "Bloco"])
df_virtual = pd.DataFrame(memory_virtual_data,
                          columns=["Endereço", "Estado", "Bloco"])

def format_size(size):
  if size < 1024:
    return f"{size} B"
  elif size < 1024 * 1024:
    return f"{size // 1024} KB"
  else:
    return f"{size // (1024 * 1024)} MB"

memory_physical_table = [["" for _ in range(32)] for _ in range(32)]

for address, block in memory_physical.items():
  if block is not None:
    row = address // 4096 // 32
    col = (address // 4096) % 32
    memory_physical_table[row][col] = format_size(len(block))

memory_physical_results = []
for address, block in memory_physical.items():
  if block is None:
    memory_physical_results.append([address, "Livre", "-"])
  else:
    memory_physical_results.append(
      [address, "Alocado", format_size(len(block))])

print("Resultados da Memória Física:")
table_headers = ["Endereço", "Estado", "Tamanho"]
table = tabulate(memory_physical_results[:25],
                 headers=table_headers,
                 tablefmt="grid")
print(table)

memory_virtual_table = [["" for _ in range(32)] for _ in range(32)]

for address, block in memory_virtual.items():
  if block is not None:
    row = address // 4096 // 32
    col = (address // 4096) % 32
    if isinstance(block, str):
      memory_virtual_table[row][col] = format_size(len(block)) + " (Física)"
    else:
      memory_virtual_table[row][col] = format_size(block['size']) + " (Swap)"

memory_virtual_results = []
for address, block in memory_virtual.items():
  if block is None:
    memory_virtual_results.append([address, "Livre", "-"])
  else:
    if isinstance(block, str):
      memory_virtual_results.append([address, "Alocado (Física)", format_size(len(block))])
    else:
      memory_virtual_results.append([address, "Alocado (Swap)", f"Posição: {block['position']}, Tamanho: {format_size(block['size'])}"])

print("\nResultados da Memória Virtual:")
table_headers = ["Endereço", "Estado", "Bloco"]
table = tabulate(memory_virtual_results[:25], headers=table_headers, tablefmt="grid")
print(table)
