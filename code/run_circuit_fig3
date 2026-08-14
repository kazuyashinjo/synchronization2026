"""Run a Trotterized XYZ-interaction circuit on a 156-qubit heavy-hex lattice.

The 176 couplings are partitioned into three parallel-execution layers. Measurement
job information for each Trotter step is saved as JSON. Configure IBM Quantum
credentials outside this source file before running it.
"""

import json
import os
from math import pi

import numpy as np
from qiskit import QuantumCircuit
from qiskit.transpiler.preset_passmanagers import generate_preset_pass_manager
from qiskit_ibm_runtime import Batch, QiskitRuntimeService, SamplerV2


backend_name = os.environ.get("QISKIT_BACKEND_NAME", "")

service = QiskitRuntimeService()
backend = service.backend(backend_name)

print(backend_name)


def add_rxyz_gate(
    circuit: QuantumCircuit,
    edge: list[int],
    theta_xx: float,
    theta_yy: float,
    theta_zz: float,
):
    qubit_a, qubit_b = edge
    circuit.cx(qubit_a, qubit_b)
    circuit.rx(-theta_xx, qubit_a)
    circuit.h(qubit_a)
    circuit.rz(-theta_zz, qubit_b)

    circuit.cx(qubit_a, qubit_b)
    circuit.s(qubit_a)
    circuit.h(qubit_a)
    circuit.rz(theta_yy, qubit_b)

    circuit.cx(qubit_a, qubit_b)
    circuit.sxdg(qubit_a)
    circuit.sx(qubit_b)


num_qubits = 156


zz_layer_1 = [
	[1,2],[4,5],[8,9],[10,11],[12,13],
	[3,16],[7,17],[15,19],
	[20,21],[22,23],[27,28],[29,30],[31,32],[33,34],
	[25,37],[38,49],
	[40,41],[42,43],[44,45],[46,47],[50,51],[53,54],
	[56,63],[57,67],[59,75],
	[60,61],[65,66],[68,69],[70,71],[72,73],
	[76,81],[79,93],
	[83,84],[85,86],[88,89],[90,91],
	[87,97],[95,99],[96,103],[98,111],
	[104,105],[106,107],[114,115],
	[101,116],[109,118],[113,119],[117,125],
	[120,121],[123,124],[128,129],[130,131],[132,133],
	[127,137],[135,139],
	[142,143],[144,145],[146,147],[150,151],[152,153]
]

zz_layer_2 = [
	[0,1],[3,4],[5,6],[7,8],[11,12],[13,14],
	[16,23],[18,31],
	[21,22],[24,25],[26,27],[32,33],[34,35],
	[29,38],[36,41],[39,53],
	[45,46],[47,48],[49,50],[51,52],[54,55],
	[43,56],[58,71],
	[62,63],[64,65],[66,67],[69,70],[74,75],
	[61,76],[73,79],[77,85],[78,89],
	[80,81],[82,83],[87,88],[91,92],[93,94],
	[97,107],[99,115],
	[101,102],[103,104],[108,109],[110,111],[112,113],
	[105,117],[116,121],
	[122,123],[125,126],[127,128],[129,130],[133,134],
	[131,138],[136,143],
	[141,142],[145,146],[147,148],[149,150],[151,152],[154,155],
]

zz_layer_3 = [
	[2,3],[6,7],[9,10],[14,15],
	[11,18],[17,27],[19,35],
	[23,24],[25,26],[28,29],[30,31],
	[21,36],[33,39],[37,45],
	[41,42],[43,44],[48,49],[52,53],
	[47,57],[51,58],[55,59],
	[61,62],[63,64],[67,68],[71,72],[73,74],
	[65,77],[69,78],
	[81,82],[84,85],[86,87],[89,90],[92,93],[94,95],
	[83,96],[91,98],
	[100,101],[102,103],[105,106],[107,108],[109,110],[111,112],[113,114],
	[118,129],[119,133],
	[121,122],[124,125],[126,127],[131,132],[134,135],
	[123,136],[137,147],[138,151],[139,155],
	[140,141],[143,144],[148,149],[153,154]
]

theta_xx = -0.25 * pi
theta_yy = -0.25 * pi
theta_zz = -0.25 * pi

with Batch(backend=backend):
    for theta_z_fraction in [0.25]:
        theta_z = theta_z_fraction * pi
        use_random_initial_phases = theta_z != 0.0

        trotter_layer = QuantumCircuit(num_qubits)
        for qubit in range(num_qubits):
            trotter_layer.rz(theta_z, qubit)

        for edge in zz_layer_1:
            add_rxyz_gate(trotter_layer, edge, theta_xx, theta_yy, theta_zz)
        for edge in zz_layer_2:
            add_rxyz_gate(trotter_layer, edge, theta_xx, theta_yy, theta_zz)
        for edge in zz_layer_3:
            add_rxyz_gate(trotter_layer, edge, theta_xx, theta_yy, theta_zz)

        pass_manager = generate_preset_pass_manager(
            optimization_level=0,
            target=backend.target,
        )

        print("theta_z=", theta_z)
        print("theta_xx=", theta_xx)
        print("theta_yy=", theta_yy)
        print("theta_zz=", theta_zz)

        num_steps = 41
        trotter_circuits = []
        for step in range(num_steps):
            trotter_circuit = QuantumCircuit(num_qubits)
            np.random.seed(19483)
            random_phase_min = 0.0 * pi
            random_phase_max = 1.0 * pi

            for qubit in range(num_qubits):
                trotter_circuit.h(qubit)
                if use_random_initial_phases:
                    random_phase = random_phase_min + (
                        random_phase_max - random_phase_min
                    ) * np.random.rand()
                    trotter_circuit.rz(random_phase, qubit)

            for _ in range(step):
                trotter_circuit = trotter_circuit.compose(trotter_layer)

            for qubit in range(num_qubits):
                trotter_circuit.h(qubit)

            trotter_circuit.measure_all()
            trotter_circuit = pass_manager.run(trotter_circuit)
            trotter_circuits.append(trotter_circuit)
            print(f"Trotter circuit with {step} Trotter steps")

        print("Gate counts:")
        gate_counts = trotter_circuits[-1].count_ops()
        for gate_name, count in gate_counts.items():
            print(f"{gate_name}: {count}")

        num_shots = 2**13
        jobs = []
        sampler = SamplerV2()
        sampler.options.default_shots = num_shots
        for circuit_index, trotter_circuit in enumerate(trotter_circuits):
            job = sampler.run([trotter_circuit])
            print(f"Job {circuit_index} ID: {job.job_id()}")
            jobs.append(job)

        metadata = {
            "backend": backend_name,
            "shots": num_shots,
            "trotter_steps": num_steps,
            "gate_counts": list(gate_counts.items()),
            "job_id": [job.job_id() for job in jobs],
        }

        output_path = (
            f"heavy_hex_156_random_phases_{use_random_initial_phases}"
            f"_theta_xx_{theta_xx / pi:.3f}pi"
            f"_theta_yy_{theta_yy / pi:.3f}pi"
            f"_theta_zz_{theta_zz / pi:.3f}pi"
            f"_theta_z_{theta_z / pi:.3f}pi.json"
        )
        with open(output_path, "w") as output_file:
            json.dump(metadata, output_file, indent=4)

        del trotter_circuit, trotter_layer, trotter_circuits, jobs
