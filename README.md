# Automated-Deadlock-Detection
import networkx as nx
import matplotlib.pyplot as plt

class DeadlockDetector:
    def __init__(self):
        self.processes = {}
        self.resources = {}

    def add_process(self, process_id):
        self.processes[process_id] = {"allocated": set(), "requested": set()}

    def add_resource(self, resource_id):
        self.resources[resource_id] = {"allocated_to": set(), "requested_by": set()}

    def allocate_resource(self, process_id, resource_id):
        self.processes[process_id]["allocated"].add(resource_id)
        self.resources[resource_id]["allocated_to"].add(process_id)

    def request_resource(self, process_id, resource_id):
        self.processes[process_id]["requested"].add(resource_id)
        self.resources[resource_id]["requested_by"].add(process_id)

    def check_deadlock(self):
        graph = nx.DiGraph()

        for process_id, process in self.processes.items():
            graph.add_node(process_id, color='blue', type='process')
            for resource_id in process["allocated"]:
                graph.add_edge(process_id, resource_id, color='black', style='solid')
            for resource_id in process["requested"]:
                graph.add_edge(resource_id, process_id, color='red', style='dashed')

        for resource_id, resource in self.resources.items():
            graph.add_node(resource_id, color='green', type='resource')

        try:
            cycles = list(nx.simple_cycles(graph))
            if cycles:
                return True
            else:
                return False
        except nx.NetworkXNoCycle:
            return False

    def draw_graph(self, show_deadlock=False):
        graph = nx.DiGraph()

        for process_id, process in self.processes.items():
            graph.add_node(process_id, color='blue', type='process')
            for resource_id in process["allocated"]:
                graph.add_edge(process_id, resource_id, color='black', style='solid')
            for resource_id in process["requested"]:
                graph.add_edge(resource_id, process_id, color='red', style='dashed')

        for resource_id, resource in self.resources.items():
            graph.add_node(resource_id, color='green', type='resource')

        pos = nx.spring_layout(graph)
        colors = [graph.nodes[n]['color'] for n in graph.nodes()]
        edge_colors = [graph[u][v]['color'] for u, v in graph.edges()]

        fig, ax = plt.subplots(figsize=(8, 6))

        node_shapes = {}
        for node in graph.nodes():
            if graph.nodes[node]['type'] == 'process':
                node_shapes[node] = 'o'
            elif graph.nodes[node]['type'] == 'resource':
                node_shapes[node] = 's'

        nx.draw(graph, pos, with_labels=True, node_color=colors, edge_color=edge_colors, node_size=2000, ax=ax,
                node_shape='o', font_size=10, font_weight='bold')

        for node, shape in node_shapes.items():
            if shape == 's':
                ax.scatter(pos[node][0], pos[node][1], s=3000, c='green', marker='s')

        plt.title("Deadlock Detection" + (" (Deadlock Present)" if show_deadlock else " (No Deadlock)"))
        plt.show()

# Example Usage - Deadlock Scenario
detector = DeadlockDetector()

detector.add_process("P1")
detector.add_process("P2")
detector.add_process("P3")

detector.add_resource("R1")
detector.add_resource("R2")
detector.add_resource("R3")
detector.add_resource("R4")

detector.allocate_resource("P1", "R1")
detector.allocate_resource("P2", "R2")
detector.allocate_resource("P3", "R3")

detector.request_resource("P1", "R2")
detector.request_resource("P2", "R3")
detector.request_resource("P3", "R1")

if detector.check_deadlock():
    detector.draw_graph(show_deadlock=True)
else:
    print("No deadlock detected!")

# Example Usage - No Deadlock Scenario
detector2 = DeadlockDetector()

detector2.add_process("P1")
detector2.add_process("P2")
detector2.add_process("P3")

detector2.add_resource("R1")
detector2.add_resource("R2")
detector2.add_resource("R3")
detector2.add_resource("R4")

detector2.allocate_resource("P1", "R1")
detector2.allocate_resource("P2", "R2")
detector2.allocate_resource("P3", "R3")

detector2.request_resource("P1", "R2")
detector2.request_resource("P2", "R3")
detector2.request_resource("P3", "R4")

if detector2.check_deadlock():
    detector2.draw_graph(show_deadlock=True)
else:
    detector2.draw_graph(show_deadlock=False)
