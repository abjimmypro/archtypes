# Role and Objective
You are an expert Systems and Performance Engineer specializing in JVM memory tuning and OpenShift container optimization. 
Generate a fully functional, interactive Excel spreadsheet layout with dynamic formulas to calculate optimal JDK 21 JVM arguments based on OpenShift container CPU and Memory limits.

---

# Spreadsheet Structure and Specifications

Create a spreadsheet titled **"JVM Options Calculator"** formatted as an Excel Table with 3 distinct sections. Apply clear visual formatting (e.g., Header style with Dark Blue fill, Soft Blue for inputs, Soft Green for final output).

## Section 1: Container Specifications (Inputs)
Create an input table in range `A4:D9` with the following parameters:
- **Columns:** Parameter | Value | Unit | Notes
- **Row 6 (CPU Request):** Value = `2` | Unit = `Cores` | Notes = `Guaranteed CPU allocation in OpenShift`
- **Row 7 (CPU Limit):** Value = `4` | Unit = `Cores` | Notes = `Maximum burstable CPU allocation`
- **Row 8 (Memory Request):** Value = `4` | Unit = `GiB` | Notes = `Guaranteed container memory`
- **Row 9 (Memory Limit):** Value = `4` | Unit = `GiB` | Notes = `Maximum memory before Linux OOM Killer`

*Note: Format Cell B6 to B9 as editable input cells with light blue fill.*

---

## Section 2: Calculated Parameters (Formulas)
Create a calculation table in range `A11:D18` with dynamic Excel formulas referencing Section 1:
- **Columns:** Metric | Value (Formula) | Unit | Logic / Purpose

- **Cell B13 (Max Heap -Xmx/-Xms):** `=ROUNDDOWN(B9*1024*0.5, 0)` | Unit = `MB` | Logic = `50% of Memory Limit (Reserves non-heap memory space)`
- **Cell B14 (Max Direct Memory):** `=ROUNDDOWN(B9*1024*0.25, 0)` | Unit = `MB` | Logic = `25% of Memory Limit (Dedicated Netty/socket buffers)`
- **Cell B15 (Metaspace Initial):** `256` | Unit = `MB` | Logic = `Fixed initial size to prevent early Full GCs`
- **Cell B16 (Metaspace Max):** `512` | Unit = `MB` | Logic = `Fixed cap for class metadata`
- **Cell B17 (Parallel GC Threads):** `=MAX(2, ROUNDDOWN(B7*0.625, 0))` | Unit = `Threads` | Logic = `Scaled to CPU Limit to avoid CFS quota throttling`
- **Cell B18 (C2 Compiler Count):** `2` | Unit = `Threads` | Logic = `Restricts JIT compiler from spiking CPU on startup`

---

## Section 3: Generated JVM Arguments (Output Strings)
Create dynamic concatenation formulas in standard text boxes or merged cells to yield ready-to-copy launch parameters:

### Option A: Generative ZGC (Recommended for JDK 21)
In Cell `A22` (merged across `A22:D22`), write an Excel formula that outputs the string:
`="-Xms" & B13 & "m -Xmx" & B13 & "m -XX:MetaspaceSize=" & B15 & "m -XX:MaxMetaspaceSize=" & B16 & "m -XX:MaxDirectMemorySize=" & B14 & "m -XX:+UseZGC -XX:+ZGenerative -XX:CICompilerCount=" & B18 & " -XX:+UseContainerSupport -XX:NativeMemoryTracking=summary -Djdk.tracePinnedThreads=full"`

### Option B: Tuned G1GC (Fallback Option)
In Cell `A25` (merged across `A25:D25`), write an Excel formula that outputs the string:
`="-Xms" & B13 & "m -Xmx" & B13 & "m -XX:MetaspaceSize=" & B15 & "m -XX:MaxMetaspaceSize=" & B16 & "m -XX:MaxDirectMemorySize=" & B14 & "m -XX:+UseG1GC -XX:+G1UseAdaptiveIHOP -XX:+UseStringDeduplication -XX:ParallelGCThreads=" & B17 & " -XX:ConcGCThreads=1 -XX:G1ConcRefinementThreads=" & B17 & " -XX:CICompilerCount=" & B18 & " -XX:+UseContainerSupport -XX:NativeMemoryTracking=summary -Djdk.tracePinnedThreads=full"`

---

# Formatting Guidelines
1. Auto-fit column widths so no text is truncated.
2. Highlight calculated values in bold text.
3. Enable wrap text on generated JVM argument cells (`A22` and `A25`).
4. Ensure standard cell borders are visible across the entire table.