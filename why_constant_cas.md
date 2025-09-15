## 为啥民航飞机爬升下降,  倾向于  恒定表速, 而不是TAS或变化表速的  垂直模式?

民航飞机在爬升和下降阶段普遍采用**恒定表速（CAS/IAS）**的垂直模式，而不是恒定真空速（TAS）或不断变化的表速，主要原因如下：

#### 失速裕度与安全管理
表速（IAS/CAS）与失速速度直接相关，能清楚地反映机翼上的气动情况，便于飞行员保持足够的失速裕度，确保安全。例如爬升阶段常规设定为“绿色区”或1.3倍失速表速（如V2+10），下降阶段也用表速管理防止过低空速导致失速。

恒定表速帮助避免由于高空真空速变大但表速变小时，误入升力不足、离失速较近的危险区域，尤其适合迭代调整机型、配置和重量影响下的界限。

#### 操纵与性能标准化
飞机失速、最大可用机动速度Vmo/Mmo等众多限速、襟翼/起落架的操作速度，全部以表速定义。恒定表速在各高度段都有直观参考，便于程序化操作和标准化流程。

利用恒定表速，可以使不同机场、不同温度和高度条件下的性能界限统一化，不同飞行阶段的推力曲线、爬升率和下降率便于计算和预测。

#### 气象与大气条件适应性
表速直接来自皮托管，自动补偿了大气密度变化，使飞行器在气温、压力波动下维持一致的气动安全裕度，而不会因高空TAS远大于IAS时导致实际升力不足。

真空速（TAS）与风无关且在高空显著提高，但此时机翼气流的实际效应只能通过IAS反映，故性能计算和风险监控紧贴表速更实用。

#### 人为操作与自动化易用性
恒定表速模式下，飞行员和自动驾驶系统只需关注单一参数即可完成垂直速度及爬升/下降率的管理，减少操作负担，提升安全。

在特殊阶段如初始爬升、着陆进近等，必须靠IAS定速，便于迅速应对异常（如发动机失效、风切变等）。

综上，恒定表速垂直模式兼顾了安全裕度、程序化管理和气动稳定性，也是民航机型统一飞行标准和法规的重要基础。

## Why Constant Indicated Airspeed Is Used in Climb and Descent
Commercial aircraft typically use a constant indicated airspeed (IAS/CAS) mode during climb and descent, rather than constant true airspeed (TAS) or a profile with varying indicated airspeed. The main reasons are:

#### Stall Margin and Safety
Indicated airspeed is directly related to stall speed and thus provides a clear indication of aerodynamic safety margins, making it easier for pilots to avoid stall conditions during climb or descent.

Maintaining constant IAS ensures consistent lift characteristics regardless of altitude, as aerodynamic performance (like stall behavior) is tied to IAS, not TAS.


#### Operational Standardization and Procedures
Aircraft limitations (stall speed, maximum operating speeds, flap/gear deployment speeds) are all defined in terms of IAS. Using constant IAS makes it straightforward to adhere to performance limits and standard operating procedures across various flight segments.

It simplifies flight management and performance calculations, as pilots and autopilot systems can rely on a single, meaningful speed reference tied directly to safety-critical behaviors.


#### Adaptation to Atmospheric Conditions
IAS is measured by the pitot-static system and naturally compensates for changes in atmospheric density. As altitude increases and air becomes thinner, the true airspeed corresponding to a given IAS increases, but aerodynamic forces remain defined by IAS.

TAS increases with altitude at fixed IAS, but the actual aerodynamic load (and risk of stall) remains unchanged, which is why IAS is preferred for vertical flight path management.


#### Human Factors and Automation
Constant IAS allows both pilots and automation to manage vertical speed and flight path with a single, intuitive speed setting, reducing workload and enhancing safety, especially during busy climb and descent phases.

During critical phases (initial climb, approach/landing), speed targets are always IAS-based to ensure safe performance margins.

Example Applications in Aviation
During climb, the autopilot is typically set to maintain a constant IAS (e.g., 250 KIAS below 10,000 feet) for optimal performance and to comply with regulatory constraints. As the aircraft climbs higher, although the IAS stays the same, the TAS increases automatically due to reduced air density.

In descent, the flight management system also targets a constant IAS for energy-efficient and predictable descent profiles, transitioning to a constant Mach number at higher altitudes for high-speed descent phases.

Pilots use published IAS references for takeoff rotation speed, climb speeds (V2, VYSE, etc.), and approach speeds—all of which provide consistent aerodynamic meaning, irrespective of altitude or temperature.

#### In summary
constant indicated airspeed during climb and descent provides standardized safety margins, simplifies crew and automation workload, and adapts naturally to changes in altitude and atmospheric conditions, making it the foundation for vertical profile management in modern commercial aviation.

