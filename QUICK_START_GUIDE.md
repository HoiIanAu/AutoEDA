# AutoEDA Quick Start Guide

## Prerequisites

Before starting, ensure you have the following installed:

- **Python 3.9+** (recommended: Python 3.11+)
- **Synopsys Design Compiler** (for commercial synthesis)
- **Cadence Innovus** (for physical implementation)
- **Yosys** (for open-source synthesis, optional)
- **Valid EDA licenses** for commercial tools

## Step 1: Environment Setup

### 1.1 Clone and Setup Repository

```bash
# Clone the repository
git clone https://github.com/Duke-CEI-Center/AutoEDA.git
cd AutoEDA

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 1.2 Configure Environment Variables

Modify the `example.env` file in the `src/` directory for EDA tool configuration:

```bash
# Synopsys Design Compiler path
EDA_SYNOPSYS_PATH=/path/to/synopsys/bin

# Cadence Innovus path
EDA_CADENCE_PATH=/path/to/cadence/bin
```

Rename the file to `.env` and it will be used automatically.

### 1.3 Verify EDA Tools

```bash
# Check Design Compiler
dc_shell -version

# Check Innovus
innovus -version

# Check open-source tools (optional)
yosys --version

```

## Step 2: Interactive Demo

### 2.1 Using the Interactive Demo Script

The easiest way to get started with AutoEDA is using the interactive demo script. This script automatically starts all services and provides a user-friendly interface for testing EDA workflows.

```bash
# Run the interactive demo
python src/demo.py
```

**What the demo script does:**
- Automatically starts all EDA servers and the intelligent client with local LLM integration  
- Provides an interactive terminal user interface 
- Demonstrates example queries and accepts your custom input
- Makes POST requests to the client API
- Shows execution results and reports

## Step 3: Start the Services

To start services manually, follow these steps:

### 3.1 Start All EDA Servers

```bash
# Start all four microservices
python3 src/run_server.py --server all
```

This command starts the 4-server microservice architecture:
- **Synthesis Service** (port 18001) - RTL to gate-level netlist (Design Compiler + Yosys)
- **Placement Service** (port 18002) - floorplan, powerplan, and placement
- **CTS Service** (port 18003) - clock tree synthesis and optimization  
- **Routing Service** (port 18004) - global/detailed routing and final save

**Alternative: Start servers individually**

```bash
# Start each service individually
python3 src/run_server.py --server synthesis --port 18001 &
python3 src/run_server.py --server placement --port 18002 &
python3 src/run_server.py --server cts --port 18003 &
python3 src/run_server.py --server routing --port 18004 &

# Wait for all services to start
sleep 5
```

### 3.2 Start AI Agent

```bash
# Start the intelligent agent (local SFT model powered)
python3 src/mcp_agent_client.py
```

### 3.3 Verify Services are Running

```bash
# Check if all ports are listening
netstat -tlnp | grep -E "(8000|1800[1-4])"

# Expected output:
# tcp        0      0 :::8000         :::*        LISTEN      [PID]/python3  (Agent)
# tcp        0      0 :::18001        :::*        LISTEN      [PID]/python3  (Synthesis)
# tcp        0      0 :::18002        :::*        LISTEN      [PID]/python3  (Placement)
# tcp        0      0 :::18003        :::*        LISTEN      [PID]/python3  (CTS)
# tcp        0      0 :::18004        :::*        LISTEN      [PID]/python3  (Routing)

# Check service health and API documentation
curl http://localhost:18001/docs  # Synthesis API docs
curl http://localhost:18002/docs  # Placement API docs
curl http://localhost:18003/docs  # CTS API docs
curl http://localhost:18004/docs  # Routing API docs
curl http://localhost:8000/docs   # AI Agent API docs
```

## Step 4: Your First Design

### 4.1 Check Available Designs

The system includes several pre-configured designs:

```bash
# List available designs
ls designs/

# Available designs:
# - des      (Verilog) - DES encryption design
# - spm      (Verilog) - OpenLane scratchpad memory design
# - b14      (VHDL)    - B14 benchmark design  
# and more!
```

### 4.2 Using AI Agent to run EDA flows

Run synthesis by making POST requests with a natural language instruction to the client:

```bash
# Test synthesis for DES design
curl -X POST http://localhost:8000/agent \
  -H "Content-Type: application/json" \
  -d '{
    "user_query": "Run synthesis for design des with 2ns clock period",
    "session_id": "demo"
  }'
```

**Expected Response:**
```json
{
  "tool_called": "synthesis",
  "tool_input": {
    "design": "des",
    "tech": "FreePDK45",
    "period": 2.0,
    "force": true,
    "vendor": "commercial"
  },
  "tool_output": {
    "status": "ok",
    "log_path": "/home/yl996/proj/mcp-eda-example/logs/synthesis/des_synthesis_20250831_143022.log",
    "reports": {
      "timing.rpt": "Timing analysis report content",
      "area.rpt": "Area utilization report content",
      "power.rpt": "Power analysis report content"
    },
    "tcl_path": "/home/yl996/proj/mcp-eda-example/result/des/FreePDK45/synthesis_complete.tcl"
  },
  "ai_reasoning": "Selected commercial synthesis with 2ns period for high-performance target",
  "suggestions": ["Consider running placement after synthesis completion"]
}
```

### 4.3 Individual Stage Execution

Besides synthesis, you can also run placement, cts and routing individually:

```bash
# Synthesis only
curl -X POST http://localhost:8000/agent \
  -H "Content-Type: application/json" \
  -d '{
    "user_query": "Run synthesis for design des with 1ns clock period using Design Compiler",
    "session_id": "demo"
  }'

# Open-source synthesis with Yosys
curl -X POST http://localhost:8000/agent \
  -H "Content-Type: application/json" \
  -d '{
    "user_query": "Run open-source synthesis for design spm using Yosys",
    "session_id": "demo"
  }'

# Placement with specific parameters
curl -X POST http://localhost:8000/agent \
  -H "Content-Type: application/json" \
  -d '{
    "user_query": "Run placement for design des with 0.8 target utilization",
    "session_id": "demo"
  }'

# Clock tree synthesis
curl -X POST http://localhost:8000/agent \
  -H "Content-Type: application/json" \
  -d '{
    "user_query": "Run CTS for design des",
    "session_id": "demo"
  }'

# Routing and final save
curl -X POST http://localhost:8000/agent \
  -H "Content-Type: application/json" \
  -d '{
    "user_query": "Run routing for design des",
    "session_id": "demo"
  }'
```

### 4.4 Complete RTL-to-GDSII Flow

To run a complete design flow:

```bash
# Complete flow for DES design with FreePDK45
curl -X POST http://localhost:8000/agent \
  -H "Content-Type: application/json" \
  -d '{
    "user_query": "Run complete EDA flow for design des with high performance optimization",
    "session_id": "demo"
  }'

# Complete flow for SPM design with open-source tools
curl -X POST http://localhost:8000/agent \
  -H "Content-Type: application/json" \
  -d '{
    "user_query": "Run complete flow for design spm using Yosys for synthesis",
    "session_id": "demo"
  }'
```

This will automatically execute:
1. **Synthesis** - RTL to gate-level netlist (commercial or open-source)
2. **Placement** - Floorplan + Powerplan + Placement
3. **CTS** - Clock tree synthesis and optimization
4. **Routing** - Global/detail routing + final GDS generation

### 4.5 Technology Support

The system supports multiple technologies:

```bash
# FreePDK45 (default) - fully supported
curl -X POST http://localhost:8000/agent \
  -H 'Content-Type: application/json' \
  -d '{
    "user_query": "Run synthesis for design des using FreePDK45 technology"
  }'

# ASAP7 
curl -X POST http://localhost:8000/agent \
  -H 'Content-Type: application/json' \
  -d '{
    "user_query": "Run synthesis for design des using ASAP7 technology with 0.5ns clock"
  }'
```

### 4.6 Session Management

The AI agent remembers your previous parameters and preferences:

```bash
# First request - specify full parameters
curl -X POST http://localhost:8000/agent \
  -H 'Content-Type: application/json' \
  -d '{
    "user_query": "Run synthesis for design des with 500MHz clock",
    "session_id": "my_session"
  }'

# Follow-up request - agent remembers design and parameters
curl -X POST http://localhost:8000/agent \
  -H 'Content-Type: application/json' \
  -d '{
    "user_query": "Now run placement with high utilization",
    "session_id": "my_session"
  }'

# Check session history
curl -X GET http://localhost:8000/session/my_session/history

# Set session preferences
curl -X POST http://localhost:8000/session/my_session/preferences \
  -H 'Content-Type: application/json' \
  -d '{
    "default_tech": "FreePDK45",
    "preferred_effort": "high",
    "target_util": 0.8
  }'
```

### 4.7 Strategy-Based Optimization

The system supports different optimization strategies:

```bash
# Fast flow (for quick verification)
curl -X POST http://localhost:8000/agent \
  -H 'Content-Type: application/json' \
  -d '{"user_query":"Run fast synthesis for design des"}'

# Performance optimization
curl -X POST http://localhost:8000/agent \
  -H 'Content-Type: application/json' \
  -d '{"user_query":"Run high performance complete flow for design des"}'

# Power optimization
curl -X POST http://localhost:8000/agent \
  -H 'Content-Type: application/json' \
  -d '{"user_query":"Run low power placement for design des"}'

# Area optimization
curl -X POST http://localhost:8000/agent \
  -H 'Content-Type: application/json' \
  -d '{"user_query":"Run small area routing for design des"}'
```

## Step 5: Direct Service API Usage

### 5.1 Synthesis Service (Port 18001)

```bash
# Commercial synthesis with Design Compiler
curl -X POST http://localhost:18001/run \
  -H 'Content-Type: application/json' \
  -d '{
    "design": "des",
    "tech": "FreePDK45",
    "period": 2.0,
    "compile_cmd": "compile_ultra",
    "power_effort": "high",
    "force": true
  }'

# Open-source synthesis with Yosys
curl -X POST http://localhost:18001/run \
  -H 'Content-Type: application/json' \
  -d '{
    "design": "spm",
    "tech": "FreePDK45",
    "vendor": "opensource",
    "period": 5.0,
    "force": true
  }'
```

### 5.2 Placement Service (Port 18002)

```bash
curl -X POST http://localhost:18002/run \
  -H 'Content-Type: application/json' \
  -d '{
    "design": "des",
    "tech": "FreePDK45",
    "flowEffort": "standard",
    "aspectRatio": 1.0,
    "place_global_timing_effort": "high",
    "force": true
  }'
```

### 5.3 CTS Service (Port 18003)

```bash
curl -X POST http://localhost:18003/run \
  -H 'Content-Type: application/json' \
  -d '{
    "design": "des",
    "tech": "FreePDK45",
    "cell_density": 0.75,
    "force": true
  }'
```

### 5.4 Routing Service (Port 18004)

```bash
curl -X POST http://localhost:18004/run \
  -H 'Content-Type: application/json' \
  -d '{
    "design": "des",
    "tech": "FreePDK45",
    "force": true
  }'
```

## Step 6: Experimental Framework

### 6.1 CodeBLEU-TCL Evaluation

The project includes a comprehensive experimental framework for evaluating TCL generation quality:

```bash
# Navigate to CodeBLEU-TCL directory
cd src/codebleu_tcl

# Run interactive evaluation demo
python3 demo/run_demo.py

# Run batch evaluation demo
python3 demo/batch_demo.py

# Run all demos
python3 demo/run_all_demos.py

# Basic evaluator test
python3 -c "
from tcl_codebleu_evaluator import TCLCodeBLEUEvaluator
evaluator = TCLCodeBLEUEvaluator()
print('✅ CodeBLEU-TCL evaluator loaded successfully')
"

# Return to project root
cd ../..
```

### 6.2 Evaluation Features

For detailed documentation on the CodeBLEU-TCL framework, see [src/codebleu_tcl/README.md](src/codebleu_tcl/README.md):

1. **EDA Command Recognition**: Supports 271+ domain-specific EDA commands
2. **Stage-Specific Weights**: Optimized evaluation for synthesis, placement, CTS, and routing
3. **Multi-Dimensional Analysis**: N-gram matching, syntax analysis, and dataflow analysis
4. **Automatic Tool Detection**: Auto-detects EDA tool type from script content

## Step 7: Monitoring and Debugging

### 7.1 Check Service Status

```bash
# Check all services are running
ps aux | grep -E "(run_server|synthesis_server|placement_server|cts_server|routing_server|mcp_agent_client)"

# Check port listening
netstat -tlnp | grep -E "(18001|18002|18003|18004|8000)"

# Test service connectivity
curl http://localhost:18001/docs  # Synthesis service
curl http://localhost:18002/docs  # Placement service  
curl http://localhost:18003/docs  # CTS service
curl http://localhost:18004/docs  # Routing service
curl http://localhost:8000/docs   # AI agent
```

### 7.2 Monitor Logs

All operations generate detailed logs in the `logs/` directory:

```
logs/
├── synthesis/          # Synthesis logs (commercial + opensource)
├── placement/          # Placement logs (floorplan + power + placement)
├── cts/               # Clock tree synthesis logs
└── routing/           # Routing logs (routing + final save)
```

Each log file contains:
- Complete command execution details
- EDA tool output and warnings
- Checkpoint file paths and status  
- Timing and resource information
- Error messages and debugging information
- Stage-specific reports and metrics

To monitor logs:
```bash
# Monitor synthesis logs
tail -f logs/synthesis/des_synthesis_*.log

# Monitor placement logs
tail -f logs/placement/des_placement_*.log

# Monitor CTS logs
tail -f logs/cts/des_cts_*.log

# Monitor routing logs
tail -f logs/routing/des_routing_*.log

# Monitor all logs simultaneously
tail -f logs/*/*.log
```

### 7.3 Check Design Results

```bash
# List available design results
ls designs/des/FreePDK45/synthesis/
ls designs/des/FreePDK45/implementation/

# Check synthesis reports
ls designs/des/FreePDK45/synthesis/*/reports/
cat designs/des/FreePDK45/synthesis/*/reports/timing.rpt

# Check implementation reports  
ls designs/des/FreePDK45/implementation/*/pnr_reports/

# Check generated TCL scripts
ls result/des/FreePDK45/

```

### 7.4 Debug Common Issues

#### Service Connection Issues
```bash
# Restart all services
pkill -f run_server.py
python3 src/run_server.py --server all

# Check individual service
curl -I http://localhost:18001/docs
```

#### Local SFT Model Issues
```bash
# Check model availability
python3 -c "
import os
model_paths = [
    'src/sft model/runs/qwen3-0.6b-20250808191946',
    'src/sft_model',
    'src/sft'
]
for path in model_paths:
    if os.path.exists(path):
        print(f'✅ Found model at: {path}')
        break
else:
    print('⚠️ No model found, using fallback')
"

# Test agent connectivity
curl -X POST http://localhost:8000/agent \
  -H 'Content-Type: application/json' \  
  -d '{"user_query":"test connection", "session_id":"debug"}'
```

#### EDA Tool Issues
```bash
# Check tool installation
which dc_shell && echo "✅ Design Compiler available"
which innovus && echo "✅ Innovus available"  
which yosys && echo "✅ Yosys available"

# Check license servers
lmstat -a | grep -E "(SERVER|FEATURE)"

# Test tool execution
dc_shell -version
innovus -version
```

#### Design-Specific Issues
```bash
# Check design files exist
ls designs/des/rtl/
ls designs/des/config.tcl

# Check clock port names match RTL
grep -r "input.*clk" designs/des/rtl/
grep "CLOCK_NAME" designs/des/config.tcl

# Verify design configuration
python3 -c "
design = 'des'
with open(f'designs/{design}/config.tcl') as f:
    config = f.read()
print(f'✅ Config for {design}:')
print(config)
"
```

## Step 8: Advanced Usage

### 8.1 Batch Processing Multiple Designs

Create a script for batch processing:

```bash
#!/bin/bash
# batch_process.sh

DESIGNS=("des" "spm" "b14" "counter")

for design in "${DESIGNS[@]}"; do
    echo "Processing design: $design"
    
    # Run synthesis
    curl -X POST http://localhost:8000/agent \
      -H "Content-Type: application/json" \
      -d "{\"user_query\":\"Run synthesis for design $design with performance optimization\", \"session_id\":\"batch_$design\"}"
    
    echo "Completed synthesis for $design"
    sleep 10  # Wait between designs
done
```

### 8.2 Adding New Designs

```bash
# Create new design directory
mkdir -p designs/your_design/rtl

# Add RTL files
cp your_rtl_files.v designs/your_design/rtl/

# Create configuration file for the design
# For Verilog designs:
cat > designs/your_design/config.tcl << EOF
set TOP_NAME "your_top_module"
set FILE_FORMAT "verilog"
set CLOCK_NAME "clk"
set clk_period 5.0
EOF

# (Alternative) For VHDL designs:
```bash
cat > designs/your_design/config.tcl << EOF
set TOP_NAME "your_top_module" 
set FILE_FORMAT "vhdl"
set CLOCK_NAME "clock"
set clk_period 5.0
EOF
```

### 8.3 Using Other Agents

You can use AutoEDA server in **VS Code Copilot** or **Cursor**. For setup instructions, refer to the [MCP SERVER SETUP GUIDE](MCP_SERVER_SETUP_GUIDE.md).

<!-- ## Next Steps

1. **Explore Different Designs**: Try running the system with all available designs (des, spm, b14, leon2, counter)
2. **Experiment with Technologies**: Test both FreePDK45 (full support) and ASAP7 (architecture ready)
3. **Try Different Tools**: Compare commercial (Design Compiler/Innovus) vs open-source (Yosys) synthesis
4. **Run Experiments**: Use the CodeBLEU-TCL evaluation framework to assess generation quality
5. **Add Custom Designs**: Create new designs following the established directory structure
6. **Customize Parameters**: Fine-tune synthesis and implementation parameters for your needs
7. **Session Management**: Leverage the AI agent's memory for complex multi-stage workflows
8. **Integration**: Integrate with your existing EDA toolchain and CI/CD pipeline -->

## Getting Help

- **API Documentation**: Check detailed [API_DOCUMENTATION.md](API_DOCUMENTATION.md) for complete reference
- **Service Health**: Use the health check endpoints to monitor system status
- **Logs**: Check service logs in the `logs/` directory for detailed execution information  
- **CodeBLEU Evaluation**: See [CodeBLEU_TCL_README.md](src/codebleu_tcl/README.md) for evaluation framework details
- **GitHub Issues**: Report problems and request features
- **Natural Language**: Use the AI agent to ask questions about the system itself
