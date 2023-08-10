// libary for front
// need methods for move+, check horizonts+ and method which will change background color of element after select
const clickedColor = 'red';
const originalColor = 'black';
class Main {
    constructor(id, name, type, role_id = null, x, y, width, height, draggable = true) {
        this.id = id;
        this.name = name;
        this.type = type;
        this.incomingRoutes = [];
        this.outgoingRoutes = [];
        this.role_id = role_id;
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
        this.draggable = draggable;
        this.isActive = false;
        this.isSelected = false;
        this.svgElement = null;
        this.prevMouseX = 0;
        this.prevMouseY = 0;
    }

    setColor(color) {
        const element = document.getElementById(this.id);
        if (element) {
            element.style.fill = color;
        }
    }
    getElementsWithRoleId(roleId) {
        return this.elements.filter((element) => element.role_id === roleId);
    }
    addIncomingRoute(route) {
        this.incomingRoutes.push(route);
    }

    addOutgoingRoute(route) {
        this.outgoingRoutes.push(route);
    }

    deleteIncomingRoute(route) {
        const index = this.incomingRoutes.indexOf(route);
        if (index !== -1) {
            this.incomingRoutes.splice(index, 1);
        }
    }

    deleteOutgoingRoute(route) {
        const index = this.outgoingRoutes.indexOf(route);
        if (index !== -1) {
            this.outgoingRoutes.splice(index, 1);
        }
    }

    move(x, y) {
        this.x = x;
        this.y = y;
        const element = document.getElementById(this.id);
        if (element) {
            element.setAttribute('x', x);
            element.setAttribute('y', y);
        }
    }

    updateRelatedElementsPosition(x, y) {
        if (this.role_id !== null) {
            const relatedElements = process.elements.filter(
                (element) => element.role_id === this.role_id && element.id !== this.id
            );

            relatedElements.forEach((element) => {
                element.move(element.x + x, element.y + y);
            });
        }
    }
}

class Roles extends Main {
    constructor(id, name, startY, height, color) {
        super(id, name, 'role', null, 0, startY, process.canvasWidth, height);
        this.startY = startY;
        this.color = color;
    }
}

class Action extends Main {
    constructor(id, name, x, y, width, height, role_id) {
        super(id, name, 'action', role_id, x, y, width, height);

    }

    // Дополнительные методы, если необходимо
}

class Switch extends Main {
    constructor(id, name, conditions, x, y, size, role_id) {
        super(id, name, 'switch', role_id, x, y, size, size); // Передаем одну размерность, так как ромб - это квадрат
        this.conditions = conditions;
        this.defaultRoute = null;
        this.routes = [];
    }

    addRoute(condition, targetActivity) {
        const route = new Route(`${this.id}-route-${this.routes.length + 1}`, this, targetActivity);
        this.routes.push({ condition, route });
    }

    setDefaultRoute(targetActivity) {
        this.defaultRoute = new Route(`${this.id}-default-route`, this, targetActivity);
    }
    setRoleId(id) {
        this.role_id = id;
    }
}

class Start extends Main {
    constructor(id, name, x, y, radius, color) {
        super(id, name, 'start', null, x, y, radius * 2, radius * 2); // Радиус умножается на 2 для получения диаметра круга
        this.color = color;
    }
}


class End extends Main {
    constructor(id, name, x, y, width, height, role_id) {
        super(id, name, 'end', x, y, width, height);
    }
}

class Route extends Main{
    constructor(id, sourceActivity, targetActivity) {
        super(id, "", 'route');
        this.sourceActivity = sourceActivity;
        this.targetActivity = targetActivity;
    }
}

class Process {
    constructor(id, canvasWidth, canvasHeight) {
        this.id = id;
        this.elements = [];
        this.roles = [];
        this.canvasWidth = canvasWidth;
        this.canvasHeight = canvasHeight;
        this.canvas = null;
        this.container = null;
        this.routes = null;
        this.actions = null;
        this.startEnd = null;
        this.swithces = null;
        this.mode = "elements";

        this.createCanvas();
        this.returnElementsOnClick();
    }

    createCanvas() {
        this.canvas = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
        this.canvas.setAttribute('id', this.id);
        this.canvas.setAttribute('width', this.canvasWidth);
        this.canvas.setAttribute('height', this.canvasHeight);
        this.canvas.style.border = '1px solid black';
        this.canvas.style.padding = '10px';
        this.routes = document.createElementNS("http://www.w3.org/2000/svg","g");
        this.routes.setAttribute('width', this.canvasWidth);
        this.routes.setAttribute('height', this.canvasHeight);
        this.role = document.createElementNS("http://www.w3.org/2000/svg","g");
        this.role.setAttribute('width', this.canvasWidth);
        this.role.setAttribute('height', this.canvasHeight);
        this.actions = document.createElementNS("http://www.w3.org/2000/svg","g");
        this.actions.setAttribute('width', this.canvasWidth);
        this.actions.setAttribute('height', this.canvasHeight);
        this.switches = document.createElementNS("http://www.w3.org/2000/svg","g");
        this.switches.setAttribute('width', this.canvasWidth);
        this.switches.setAttribute('height', this.canvasHeight);
        this.startEnd = document.createElementNS("http://www.w3.org/2000/svg","g");
        this.startEnd.setAttribute('width', this.canvasWidth);
        this.startEnd.setAttribute('height', this.canvasHeight);

    }
    getElementsAtPoint(x, y) {
        const elementsAtPoint = [];
        this.elements.forEach((element) => {
            const elementX = element.x || 0;
            const elementY = element.y || 0;
            const elementWidth = element.width || 0;
            const elementHeight = element.height || 0;

            if (x >= elementX && x <= elementX + elementWidth && y >= elementY && y <= elementY + elementHeight) {
                if (element.isSelected == false) {
                    element.isSelected = true;
                } else if (element.isSelected == true) {
                    element.isSelected = false;
                }
                this.elements.forEach((el) => {
                    el.isActive = false;
                });
                element.isActive = true;
                elementsAtPoint.push(element);
            }
        });

        return elementsAtPoint;
    }
    addElement(element) {
        this.elements.push(element);
    }

    deleteElement(element) {
        const index = this.elements.indexOf(element);
        if (index !== -1) {
            this.elements.splice(index, 1);
            this.renderElements();
        }
    }
    returnElementsOnClick() {
        this.canvas.addEventListener('click', (event) => {
            const boundingRect = this.canvas.getBoundingClientRect();
            const clickX = event.clientX - boundingRect.left;
            const clickY = event.clientY - boundingRect.top;


            const elementsAtClick = this.getElementsAtPoint(clickX, clickY);
            //console.log(elementsAtClick);
            // Выводим элемент на консоль
            elementsAtClick.forEach((element) => {
                console.log('Элемент:', element);
            });
        });
    }
    selectElement() {
        this.canvas.addEventListener('click', (event) => {

        });
    }
    changeMode(type) {
        this.mode = type;
    }
    createRole(name, startY, height, color) {
        const width = this.canvasWidth;
        const role = new Roles(`${this.id}-role-${this.roles.length + 1}`, name, startY, height, color);
        this.roles.push(role);
        this.addElement(role);
        return role;
    }

    deleteRole(role) {
        const index = this.roles.indexOf(role);
        if (index !== -1) {
            this.roles.splice(index, 1);
            this.deleteElement(role);
        }
    }

    createAction(name, x, y, width, height, role_id) {
        const action = new Action(`${this.id}-action-${this.elements.length + 1}`, name, x, y, width, height, role_id);
        this.addElement(action);
        return action;
    }

    // Метод для нахождения элемента Role по его role_id
    findRoleById(roleId) {
        return this.roles.find((role) => role.id === roleId);
    }

    // Метод для нахождения связанных элементов Action по role_id
    findActionsByRoleId(roleId) {
        return this.elements.filter((element) => element instanceof Action && element.role_id === roleId);
    }

    createSwitch(name, conditions, x, y, width, height, role_id) {
        const switchElement = new Switch(`${this.id}-switch-${this.elements.length + 1}`, name, conditions, x, y, width, height, role_id);
        switchElement.setRoleId(role_id);
        this.addElement(switchElement);
        return switchElement;
    }

    createStart(name, x, y, radius, color, role_id) {
        const startElement = new Start(`${this.id}-start-${this.elements.length + 1}`, name, x, y, radius, color, role_id);
        this.addElement(startElement);
        return startElement;
    }

    createEnd(name, x, y, width, height, role_id) {
        const endElement = new End(`${this.id}-end-${this.elements.length + 1}`, name, x, y, width, height, role_id);
        this.addElement(endElement);
        return endElement;
    }
    createArrow(fromElement, toElement) {
        const route = new Route (`${this.id}-route-${this.elements.length + 1}`, fromElement, toElement);
        this.addElement(route);
        this.renderArrow();
        return route;
    }
    checkPosition(x, y) {
        let flag = false;
        this.elements.forEach((element) => {
           if (element.type == 'action' || element.type == 'start' || element.type == 'end') {
               if (x >= element.x && x <= element.x + element.width && y >= element.y && y <= element.y + element.height)
                   flag = true;
           } else if (element.type == 'switch') {

           }
        });
        return flag;
    }
    arrow() {
        this.elements.forEach((element) => {
            if (element.type != 'route')
                return;
            const from = element.sourceActivity;
            const to = element.targetActivity;
            let xf = from.x, yf = from.y, xt = to.x, yt = to.y;
            let xp = [], yp = [], step = 10;


        });
    }
    renderArrow() {
        this.routes.innerHTML = '';
        this.elements.forEach((element) => {

            if (element.type != 'route') {
                return;
            }
            const from = element.sourceActivity;
            const to = element.targetActivity;
            if(!from.outgoingRoutes.includes(element.id)) {
                from.outgoingRoutes.push(element.id);
            }
            if (!from.incomingRoutes.includes(element.id)) {
                to.incomingRoutes.push(element.id);
            }
            let svgElement = document.createElementNS('http://www.w3.org/2000/svg', 'line');
            let svgElementB = document.createElementNS('http://www.w3.org/2000/svg', 'line');
            let xFrom = from.x;
            let yFrom = from.y;
            let xTo = to.x;
            let yTo = to.y;
            if (to.type == 'switch') {
                if (Math.abs(yFrom + from.height/2 - yTo) < 1){
                    this.setLinePoints(svgElement, xFrom + from.width, yFrom + from.height/2, xTo, yTo);
                    svgElement.id = element.id;
                    svgElement.classList.add('bpmn-element');
                    svgElement.classList.add(`bpmn-${element.type}`);
                } else if (yFrom + from.height/2 < yTo) {
                    this.setLinePoints(svgElement, xFrom + from.width/2, yFrom + from.height,
                        xFrom + from.width/2, yTo);
                    this.setLinePoints(svgElementB, xFrom + from.width/2, yTo, xTo, yTo);
                    svgElement.id = element.id;
                    svgElement.classList.add('bpmn-element');
                    svgElement.classList.add(`bpmn-${element.type}`);
                    svgElementB.id = element.id;
                    svgElementB.classList.add('bpmn-element');
                    svgElementB.classList.add(`bpmn-${element.type}`);
                } else if (yFrom + from.height/2 > yTo) {
                    this.setLinePoints(svgElement, xFrom + from.width/2, yFrom,
                        xFrom + from.width / 2, yTo);
                    this.setLinePoints(svgElementB, xFrom + from.width / 2, yTo,
                        xTo, yTo);
                    svgElement.id = element.id;
                    svgElement.classList.add('bpmn-element');
                    svgElement.classList.add(`bpmn-${element.type}`);
                    svgElementB.id = element.id;
                    svgElementB.classList.add('bpmn-element');
                    svgElementB.classList.add(`bpmn-${element.type}`);
                }
            } else {
                if (from.type == 'start') {
                    xFrom = from.x - from.width / 2;
                    yFrom = from.y - from.height / 2;
                }
                if (to.type == 'start') {
                    xTo = to.x - to.width / 2;
                    yTo = to.y - to.height / 2;
                }
                if (Math.abs(xFrom - xTo) < 1) {
                    this.setLinePoints(svgElement, xFrom + from.width / 2, secondPoint(yFrom, yFrom + from.height, yTo, yTo + to.height),
                        xTo + to.width / 2, thirdPoint(yFrom, yFrom + from.height, yTo, yTo + to.height));
                    svgElement.id = element.id;
                    svgElement.classList.add('bpmn-element');
                    svgElement.classList.add(`bpmn-${element.type}`);
                } else if (Math.abs(yFrom - yTo) < 1) {
                    this.setLinePoints(svgElement, secondPoint(xFrom, xFrom + from.width, xTo, xTo + to.width), yFrom + from.height / 2,
                        thirdPoint(xFrom, xFrom + from.width, xTo, xTo + to.width), yTo + to.height / 2);
                    svgElement.id = element.id;
                    svgElement.classList.add('bpmn-element');
                    svgElement.classList.add(`bpmn-${element.type}`);
                } else if (xFrom < xTo && yFrom < yTo) {
                    this.setLinePoints(svgElement, xFrom + from.width, yFrom + from.height / 2,
                        xTo + to.width / 2, yFrom + from.height / 2);
                    svgElement.id = element.id;
                    svgElement.classList.add('bpmn-element');
                    svgElement.classList.add(`bpmn-${element.type}`);
                    this.setLinePoints(svgElementB, xTo + to.width / 2, yFrom + from.height / 2,
                        xTo + to.width / 2, yTo);
                    svgElementB.id = element.id;
                    svgElementB.classList.add('bpmn-element');
                    svgElementB.classList.add(`bpmn-${element.type}`);
                } else if (xFrom < xTo && yFrom > yTo) {
                    this.setLinePoints(svgElement, xFrom + from.width, yFrom + from.height / 2,
                        xTo + to.width / 2, yFrom + from.height / 2);
                    svgElement.id = element.id;
                    svgElement.classList.add('bpmn-element');
                    svgElement.classList.add(`bpmn-${element.type}`);
                    this.setLinePoints(svgElementB, xTo + to.width / 2, yFrom + from.height / 2,
                        xTo + to.width / 2, yTo + to.height);
                    svgElementB.id = element.id;
                    svgElementB.classList.add('bpmn-element');
                    svgElementB.classList.add(`bpmn-${element.type}`);
                } else if (xFrom > xTo && yFrom < yTo) {
                    this.setLinePoints(svgElement, xFrom, yFrom + from.height / 2, xTo + to.width / 2, yFrom + from.height / 2);
                    svgElement.id = element.id;
                    svgElement.classList.add('bpmn-element');
                    svgElement.classList.add(`bpmn-${element.type}`);
                    this.setLinePoints(svgElementB, xTo + to.width / 2, yFrom + from.height / 2, xTo + to.width / 2, yTo);
                    svgElementB.addEventListener('mousedown', dragStart);
                    svgElementB.id = element.id;
                    svgElementB.classList.add('bpmn-element');
                    svgElementB.classList.add(`bpmn-${element.type}`);
                } else if (xFrom > xTo && yFrom > yTo) {
                    this.setLinePoints(svgElement, xFrom, yFrom + from.height / 2, xTo + to.width / 2, yFrom + from.height / 2);
                    svgElement.id = element.id;
                    svgElement.classList.add('bpmn-element');
                    svgElement.classList.add(`bpmn-${element.type}`);
                    this.setLinePoints(svgElementB, xTo + to.width / 2, yFrom + from.height / 2,
                        xTo + to.width / 2, yTo + to.height);
                    svgElementB.id = element.id;
                    svgElementB.classList.add('bpmn-element');
                    svgElementB.classList.add(`bpmn-${element.type}`);
                }
            }
        });
    }
    setLinePoints(svgElement, a, b, c, d) {
        svgElement.setAttribute('x1', a);
        svgElement.setAttribute('y1', b);
        svgElement.setAttribute('x2', c);
        svgElement.setAttribute('y2', d);
        svgElement.setAttribute('stroke', 'black');
        svgElement.setAttribute('stroke-width', '5');
        svgElement.addEventListener('mousedown', dragStart);
        this.routes.appendChild(svgElement);
    }
    switchToElement(toElement) {
        let route = document.getElementById(toElement.incomingRoutes[0]);

    }
    enableElementClickHandlers(clickCallback, unclickCallback) {
        this.elements.forEach((element) => {
            const svgElement = document.getElementById(element.id);
            if (!svgElement)
                return;
            svgElement.addEventListener('click', () => {
                if (element.clicked) {
                    element.clicked = false;
                    unclickCallback(element);
                } else {
                    element.clicked = true;
                    clickCallback(element);
                }
            });
        });

    }

    renderElements() {
        this.canvas.innerHTML = ''; // Очищаем холст
        this.elements.forEach((element) => {
            let svgElement;

            if (element.type === 'switch') {
                svgElement = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
                const size = element.width || 0;

                if (size) {
                    const x = element.x || 0;
                    const y = element.y || 0;

                    const points = [
                        `${x},${y}`,
                        `${x + size},${y - size}`,
                        `${x + size},${y + size}`,
                    ].join(' ');

                    svgElement.setAttribute('points', points);
                    svgElement.setAttribute('draggable', true);
                    svgElement.addEventListener('mousedown', dragStart);

                }
            } else if (element.type === 'action') {
                svgElement = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
                const x = element.x || 0;
                const y = element.y || 0;

                if (element.width && element.height) {
                    svgElement.setAttribute('x', x);
                    svgElement.setAttribute('y', y);
                    svgElement.setAttribute('width', element.width);
                    svgElement.setAttribute('height', element.height);
                    svgElement.setAttribute('draggable', true);
                    svgElement.setAttribute('fill', 'rgb(255, 255, 255)');
                    svgElement.addEventListener('mousedown', dragStart);
                }
                //this.actions.appendChild(element);
            } else if (element.type === 'role') {
                svgElement = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
                const x = element.x || 0;
                const y = element.startY || 0; // Используйте startY для задания стартовой высоты строки

                if (element.width && element.height) {
                    svgElement.setAttribute('x', x);
                    svgElement.setAttribute('y', y);
                    svgElement.setAttribute('width', element.width);
                    svgElement.setAttribute('height', element.height);
                    svgElement.setAttribute('draggable', true);
                    svgElement.setAttribute('fill', element.color); // Установите цвет элемента
                    svgElement.addEventListener('mousedown', dragStart);
                }
            } else if (element.type === 'start') {
                svgElement = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
                const radius = element.width / 2;
                const cx = element.x; // Координата центра круга по оси X
                const cy = element.y; // Координата центра круга по оси Y (используйте startY для задания стартовой высоты строки)
                svgElement.setAttribute('cx', cx);
                svgElement.setAttribute('cy', cy);
                svgElement.setAttribute('r', radius);
                svgElement.setAttribute('draggable', true);
                svgElement.setAttribute('fill', element.color); // Установите цвет элемента
                //svgElement.addEventListener('mousedown', dragStart);
            }
            if (element.type === 'role') {
                svgElement.id = element.id;
                svgElement.classList.add('bpmn-element');
                svgElement.classList.add(`bpmn-${element.type}`);
                this.role.appendChild(svgElement);
            }
            if (element.type === 'action') {
                svgElement.id = element.id;
                svgElement.classList.add('bpmn-element');
                svgElement.classList.add(`bpmn-${element.type}`);
                this.actions.appendChild(svgElement);
            }
            if (element.type === 'switch') {
                svgElement.id = element.id;
                svgElement.classList.add('bpmn-element');
                svgElement.classList.add(`bpmn-${element.type}`);
                this.switches.appendChild(svgElement);
            }
            if (element.type === 'start' || element.type === 'end') {
                svgElement.id = element.id;
                svgElement.classList.add('bpmn-element');
                svgElement.classList.add(`bpmn-${element.type}`);
                this.startEnd.appendChild(svgElement);
            }

        });

        this.canvas.appendChild(this.role);
        this.canvas.appendChild(this.switches);
        this.canvas.appendChild(this.actions);
        this.canvas.appendChild(this.startEnd);
        this.canvas.appendChild(this.routes);

    }

}
let offset = { x: 0, y: 0 };

function dragStart(event) {
    dragElement = event.target;
    if (!dragElement) {
        return;
    }
    dragElement.style.opacity = '0.5';

    let rect = dragElement.getBoundingClientRect();
    offset.x = event.clientX - rect.left;
    offset.y = event.clientY - rect.top;

    document.addEventListener('mousemove', dragMove);
    document.addEventListener('mouseup', dragEnd);
}

// Drag move event
function dragMove(event) {
    if (process.mode === "elements") {
        dragElement = event.target;
        let x = event.clientX - offset.x;
        let y = event.clientY - offset.y;
        if (!dragElement) {
            return;
        } else {
            const elementId = dragElement.id;
            const element = process.elements.find((elem) => elem.id === elementId);
            if (element) {
                const minX = 0;
                const minY = 0;
                const maxX = process.canvasWidth - element.width;
                const maxY = process.canvasHeight - element.height;
                if (element instanceof Roles) {
                    // Элемент класса Role, проверяем столкновения только с другими элементами класса Role
                    const [newX, newY] = checkCollisionForRoles(element, x, y);
                    x = Math.max(minX, Math.min(newX, maxX)); // Ограничиваем координаты x, чтобы элемент не выходил за пределы холста
                    y = Math.max(minY, Math.min(newY, maxY)); // Ограничиваем координаты y, чтобы элемент не выходил за пределы холста
                    for (let k in process.elements) {
                        if (element.id == process.elements[k].role_id) {
                            let oldX = document.getElementById(process.elements[k].id).x.animVal.value;
                            process.elements[k].move(oldX, y);
                        }
                    }

                } else if (element instanceof Switch) {

                    let deltaX = x - element.x;
                    let deltaY = y - element.y;

                    // Move the polygon by adjusting its points
                    let points = dragElement.getAttribute('points').split(' ');
                    let updatedPoints = points.map(function (point) {
                        let coords = point.split(',');
                        let newX = parseFloat(coords[0]) + deltaX;
                        let newY = parseFloat(coords[1]) + deltaY;
                        return newX + ',' + newY;
                    });
                    dragElement.setAttribute('points', updatedPoints.join(' '));

                    // Передвигаем элементы, привязанные к данной строке
                    for (const el of process.elements) {
                        if (el.role_id === element.role_id) {
                            el.move(el.x + deltaX, el.y + deltaY);
                        }
                    }
                    process.renderArrow();
                } else if (element instanceof Action) {
                    // For rectangles and areas

                    dragElement.setAttribute('x', x);
                    dragElement.setAttribute('y', y);
                    process.renderArrow();
                    let roleOfAction = process.findRoleById(element.role_id);

                    if (roleOfAction) {
                        if ((y > roleOfAction.startY + roleOfAction.height) || (y + element.height) < roleOfAction.startY) {
                            element.role_id = "null";
                            console.log('1');
                        }
                    }
                } else if (element instanceof Start) {
                    // For circles
                    const radius = element.width / 2 || 0;
                    x = event.clientX - offset.x + radius || 0;
                    y = event.clientY - offset.y + radius || 0;

                }
                element.move(x, y);
            }

        }
    }

}

function checkCollisionForRoles(currentElement, x, y) {
    let collidingElement = null;

    for (const element of process.elements) {
        // Пропускаем проверку себя и элементов других классов
        if (element === currentElement || !(element instanceof Roles)) continue;

        // Проверяем столкновение по координатам
        if (
            x < element.x + element.width &&
            x + currentElement.width > element.x &&
            y < element.y + element.height &&
            y + currentElement.height > element.y
        ) {
            // Если найдено столкновение, находим ближайшую свободную позицию
            const distanceX = element.x - x - currentElement.width;
            const distanceY = element.y - y - currentElement.height;

            if (Math.abs(distanceX) < Math.abs(distanceY)) {
                // Перемещаем элемент по оси X
                if (distanceX > 0) {
                    // Текущий элемент находится левее столкнувшегося элемента, сдвигаем вправо
                    x = element.x + element.width;
                } else {
                    // Текущий элемент находится правее столкнувшегося элемента, сдвигаем влево
                    x = element.x - currentElement.width;
                }
            } else {
                // Перемещаем элемент по оси Y
                if (distanceY > 0) {
                    // Текущий элемент находится выше столкнувшегося элемента, сдвигаем вниз
                    y = element.y + element.height;
                } else {
                    // Текущий элемент находится ниже столкнувшегося элемента, сдвигаем вверх
                    y = element.y - currentElement.height;
                }
            }
        }
    }

    return [x, y]; // Возвращаем обновленные координаты для элемента в виде массива [x, y]
}
// Drag end event
function dragEnd(event) {
    dragElement.style.opacity = '1';
    dragElement = null;

    document.removeEventListener('mousemove', dragMove);
    document.removeEventListener('mouseup', dragEnd);

}
function secondPoint(a, b, c, d) {
    let arr = [];
    arr.push(a);
    arr.push(b);
    arr.push(c);
    arr.push(d);
    arr.sort(function (s, k) {
        return s - k;
    });
    return arr[1];
}
function thirdPoint(a, b, c, d) {
    let arr = [];
    arr.push(a);
    arr.push(b);
    arr.push(c);
    arr.push(d);
    arr.sort(function (s, k) {
        return s - k;
    });
    return arr[2];
}
// Функция для обработки клика на элементе
function handleElementClick(element) {
    const svgElement = document.getElementById(element.id);
    svgElement.setAttribute('fill', clickedColor);
}

// Функция для обработки второго клика на элементе
function handleElementUnclick(element) {
    const svgElement = document.getElementById(element.id);
    svgElement.setAttribute('fill', originalColor);
}
function getMousePosition(evt) {
    const CTM = process.canvas.getScreenCTM();
    return {
        x: (evt.clientX - CTM.e) / CTM.a,
        y: (evt.clientY - CTM.f) / CTM.d
    };
}

// Включаем обработчики клика на элементы

// Создаем холст для отображения элементов
const process = new Process('process', 800, 600);
//const role1 = process.createRole('Role1', 100, 150, 'lightblue');
//const role2 = process.createRole('Role2', 300, 50, 'lightgreen');
const action1 = process.createAction('Action 1', 50, 250, 100, 50);
const action2 = process.createAction('Action 2', 450, 100, 100, 50);
//const action3 = process.createAction('Action 3', 50, 150, 100, 50, role2.id);
//const start = process.createStart('Start', 300, 350, 20, 'yellow');
const diamond = process.createSwitch('switch1', '2323', 350, 100, 30, 30);
const arrow1 = process.createArrow(diamond,action2);
const arrow = process.createArrow(action1, diamond);

process.renderElements();
document.body.appendChild(process.canvas);
process.enableElementClickHandlers(handleElementClick, handleElementUnclick);






// Надо добавить отрисовку стрелок между элементами
